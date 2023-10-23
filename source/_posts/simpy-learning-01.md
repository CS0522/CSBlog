---
title: 【学习笔记】SimPy 学习（一）：基础
tags:
  - SimPy
  - Python
toc: true
languages:
  - zh-CN
categories: 
  - 学习笔记
comments: false
cover: false
date: 2023-10-23 21:00:52
---

学习 Python 仿真库 SimPy

<!-- more -->

## SimPy 简介
> "SimPy 是基于过程的离散事件的标准 Python 模拟框架"  
> [SimPy 文档](https://simpy.readthedocs.io/en/latest/)  

## SimPy 安装
```bash
pip install simpy
```

## SimPy 核心概念

SimPy 是离散事件驱动的仿真库。所有活动部件，例如车辆、顾客，即便是信息，都可以用 `process` 来模拟。这些 `process` 存放在 `environment` 中。所有 `process` 之间，以及与 `environment` 之间的互动，通过 `event` 来进行；`process` 表达为 `generators`，构建 `event` 并通过 `yield` 语句抛出事件；当一个进程抛出事件，进程会被暂停，直到事件被 `triggered`。多个进程可以等待同一个事件。 SimPy 会按照这些进程抛出的事件 `triggered` 的先后， 来恢复进程。

其中最重要的一类事件是 `Timeout`，这类事件允许一段时间后再被激活，用来表达一个进程休眠或者保持当前的状态持续指定的一段时间。这类事件通过 `Environment.timeout` 来调用。

个人理解：
* `process` = 函数
* `environment` = 类（对象）
* `run` = 开始执行实例的函数
* `event` = if/else 语句
* `timeout` = sleep()
* `until` = 循环结束条件
* `yield` = trigger

### Environment
决定仿真的起点和终点，管理仿真元素之间的关联

#### APIs
* 添加仿真进程：`simpy.Environment.process()`
* 创建事件： `simpy.Environment.event()`
* 提供延时事件：`simpy.Environment.timeout()`
* 仿真结束的条件：`until=xxx`
* 仿真启动：`simpy.Environment.run()`
* 现在的时间：`simpy.Environment.now`

#### Example 1
<details>
<summary>定义一个进程，仿真的简单代码结构</summary>

```python
import simpy

# 定义一个汽车进程
def car(env):
    while True:
        print('Start parking at %d' % env.now)
        parking_duration = 5
        # 进程延时 5s
        yield env.timeout(parking_duration) 
        print('Start driving at %d' % env.now)
        trip_duration = 2
        # 延时 2s
        yield env.timeout(trip_duration) 

# 仿真启动
env = simpy.Environment()   # 实例化环境
env.process(car(env))   # 添加汽车进程
env.run(until = 15)   # 设定仿真结束条件, 这里是 15s 后停止
```
</details>

#### Example 2
<details>
<summary>描述一个汽车驾驶一段时间后停车充电， 汽车驾驶进程和电池充电进程通过事件的激活来相互影响</summary>

```python
"""
描述一个汽车驾驶一段时间后停车充电， 汽车驾驶进程和电池充电进程通过事件的激活来相互影响
"""

import simpy

from random import seed, randint
seed(23)

class ENV:
    def __init__(self, env):
        self.env = env
        self.drive_proc = env.process(self.drive(env))
        self.bat_ctrl_proc = env.process(self.bat_ctrl(env))
        # 这里的 reactivate 和 sleep 是先执行了 drive() 和 bat_ctrl() 中的
        # ？下面这段在做什么
        self.bat_ctrl_reactivate = env.event()
        self.bat_ctrl_sleep = env.event()

    # 驾驶进程
    def drive(self, env):
        while True:
            # drive 20~40 minutes
            print("Start driving at: ", env.now)
            yield env.timeout(randint(20, 40))
            print("End driving at: ", env.now)

            # parking 1~6 hours
            print("Start parking at: ", env.now)
            # activate battery charging
            # 这段代码应该是指接收信号
            self.bat_ctrl_reactivate.succeed()
            self.bat_ctrl_reactivate =  env.event()

            yield env.timeout(randint(60, 360)) & self.bat_ctrl_sleep
            print("End parking at: ", env.now)

    # 电池充电进程
    def bat_ctrl(self, env):
        while True:
            print("Charge sleep at: ", env.now)
            yield self.bat_ctrl_reactivate
            print("Charge activate at: ", env.now)
            yield env.timeout(randint(30, 90))
            print("Charge end at:", env.now)
            # 这段代码应该是指接收信号
            self.bat_ctrl_sleep.succeed()
            self.bat_ctrl_sleep = env.event()


if __name__ == "__main__":
    env = simpy.Environment()
    ENV(env)
    env.run(until=300)
```
</details>


### Resource / Store
仿真中涉及的人力资源以及工艺上的物料消耗都会抽象用 `Resource` 来表达，主要的 `method` 是 `request`。`Store` 处理各种优先级的队列问题，表现跟 `queue` 一致，通过 `get/put` 存放 `item`。

#### APIs
* 资源或限制条件：`simpy.Resource(env, capacity)`
* 优先级、不可打断正在服务的进程：`simpy.PriorityResource`
* 优先级、可以打断正在服务的进程：`simpy.PreemptiveResource`
* 存取 `Item`、遵循先来后到：`simpy.Store`
* 添加优先级：`simpy.PriorityStore`
* 存在分类：`simpy.FilterStore`
* 连续不可分的元素：`simpy.Container`

#### Example 3
<details>
<summary>银行排队服务例子：一个柜台对客户进行服务，服务耗时，客户等候过长会离开柜台</summary>

```python
"""
银行排队服务例子：一个柜台对客户进行服务，服务耗时，客户等候过长会离开柜台
"""

import simpy
import random

RANDOM_SEED = 42
NEW_CUSTOMERS = 5  # 客户数
INTERVAL_CUSTOMERS = 10.0  # 客户到达的间距时间
MIN_PATIENCE = 1  # 客户等待时间, 最小
MAX_PATIENCE = 3  # 客户等待时间, 最大

# 生成客户
def source(env, number, interval, counter):
    for i in range(number):
        c = customer(env, 'Customer%02d' % i, counter, time_in_bank = 12.0)
        env.process(c)
        t = random.expovariate(1.0 / interval)
        yield env.timeout(t)

# 客户到达、服务、离开
def customer(env, name, counter, time_in_bank):
    arrive = env.now
    print('%7.4f %s: Here I am' % (arrive, name))

    with counter.request() as req:
        patience = random.uniform(MIN_PATIENCE, MAX_PATIENCE)
        # 等待柜员服务或者超出忍耐时间离开队伍
        results = yield req | env.timeout(patience)
        wait = env.now - arrive
        if req in results:
        # 到达柜台
            print('%7.4f %s: Waited %6.3f' % (env.now, name, wait))
            tib = random.expovariate(1.0 / time_in_bank)
            yield env.timeout(tib)
            print('%7.4f %s: Finished' % (env.now, name))
        else:
            # 没有服务到位
            print('%7.4f %s: RENEGED after %6.3f' % (env.now, name, wait))

# Setup and start the simulation
print('Bank renege')
random.seed(RANDOM_SEED)
env = simpy.Environment()

# Start processes and run
counter = simpy.Resource(env, capacity = 1)
env.process(source(env, NEW_CUSTOMERS, INTERVAL_CUSTOMERS, counter))
env.run()
```