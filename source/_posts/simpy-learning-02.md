---
title: 【学习笔记】SimPy 学习（二）：进程交互与资源共享
tags:
  - SimPy
  - Python
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - SimPy
comments: false
cover: false
date: 2023-10-24 08:13:23
---

学习 SimPy 的一些核心概念，分为 Environment 的进程交互和 Resource 的资源共享

<!-- more -->

## Environment
决定仿真的起点和终点，管理仿真元素之间的关联

### APIs
* 添加仿真进程：`simpy.Environment.process()`
* 创建事件： `simpy.Environment.event()`
* 提供延时事件：`simpy.Environment.timeout()`
* 仿真结束的条件：`until=xxx`
* 仿真启动：`simpy.Environment.run()`
* 现在的时间：`simpy.Environment.now`

### Example 2.1
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

### Example 2.2
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
            # 这段代码应该是指创建事件
            self.bat_ctrl_reactivate.succeed()
            self.bat_ctrl_reactivate =  env.event()

            # 超时并挂起，等待 bat_ctrl_sleep 事件发生
            yield env.timeout(randint(60, 360)) & self.bat_ctrl_sleep
            print("End parking at: ", env.now)

    # 电池充电进程
    def bat_ctrl(self, env):
        while True:
            print("Charge sleep at: ", env.now)
            # 挂起。接收到 drive() 中的事件发生后恢复
            yield self.bat_ctrl_reactivate
            print("Charge activate at: ", env.now)
            yield env.timeout(randint(30, 90))
            print("Charge end at:", env.now)
            # 这段代码应该是指创建事件
            self.bat_ctrl_sleep.succeed()
            self.bat_ctrl_sleep = env.event()


if __name__ == "__main__":
    env = simpy.Environment()
    ENV(env)
    env.run(until=300)
```
</details>

汽车的进程需要引用环境（`env`）来创建新的事件。汽车的行为被描述成一个无限循环。记住，这个函数是一个生成器。尽管它永远不会终止，但一旦到达 `yield` 语句，它会将控制流传回仿真程序。触发生成的事件后（`事件发生`），模拟将在此语句中恢复该函数。

<details>
<summary>查看结果</summary>

```bash
Start driving at:  0
Charge sleep at:  0
End driving at:  29
Start parking at:  29
Charge activate at:  29
Charge end at: 60
Charge sleep at:  60
End parking at:  131
Start driving at:  131
End driving at:  169
Start parking at:  169
Charge activate at:  169
Charge end at: 226
Charge sleep at:  226
```
</details>

### Example 2.3
<details>
<summary>在电动汽车还在充电过程中，接到一个紧急通知，需要中断充电进程马上出门</summary>

```python
import simpy
 
def driver(env, car):
    yield env.timeout(3)
    car.action.interrupt()

class Car:
    def __init__(self, env):
        self.env = env
        # 实例化时开始run进程
        self.action = env.process(self.run())
        
    def run(self):
        while True:
            print('开始停车充电： %d' % self.env.now)
            charge_duration = 5
            # 挂起process()返回的进程
            # 等待充电结束
            try:
                yield self.env.process(self.charge(charge_duration))
            except simpy.Interrupt:
                # 当我们接收到一个中断，我们停止充电，切换到行驶状态
                print('终止充电。祈祷电量足够......')
                
            # 充电结束，可以重新上路
            print('开始行驶： %d' % self.env.now)
            trip_duration = 2
            yield self.env.timeout(trip_duration)
 
    def charge(self, duration):
        yield self.env.timeout(duration)  

if __name__ == "__main__":
    env = simpy.Environment()
    car = Car(env)
    env.process(driver(env, car))
```
</details>

<details>
<summary>查看结果</summary>

```bash
开始停车充电： 0
终止充电。祈祷电量足够......
开始行驶： 3
开始停车充电： 5
开始行驶： 10
开始停车充电： 12
```
</details>


## Resource / Store
仿真中涉及的人力资源以及工艺上的物料消耗都会抽象用 `Resource` 来表达，主要的 `method` 是 `request`。`Store` 处理各种优先级的队列问题，表现跟 `queue` 一致，通过 `get/put` 存放 `item`。

### APIs
* 资源或限制条件：`simpy.Resource(env, capacity)`
* 优先级、不可打断正在服务的进程：`simpy.PriorityResource`
* 优先级、可以打断正在服务的进程：`simpy.PreemptiveResource`
* 存取 `Item`、遵循先来后到：`simpy.Store`
* 添加优先级：`simpy.PriorityStore`
* 存在分类：`simpy.FilterStore`
* 连续不可分的元素：`simpy.Container`

### Example 2.4
<details>
<summary>电动汽车使用两个充电桩中间的一个进行充电。如果这两个点都在使用，汽车将等待其中一个点直至其再次可用，然后开始给充电，完成后离开充电站</summary>

```python
import simpy

def car(env, name, bcs, driving_time, charge_duration):
    # 驶向充电站
    yield env.timeout(driving_time)

    print('%s 到达时间 %d' % (name, env.now))
    # 请求充电桩
    with bcs.request() as req:
        yield req

        print('%s 充电开始时间 %d' % (name, env.now))
        yield env.timeout(charge_duration)
        print('%s 充电结束并驶离时间 %d' % (name, env.now))


env = simpy.Environment()
# bcs 充电桩资源
bcs = simpy.Resource(env, capacity = 2)
# 创建汽车进程
for i in range(4):
    env.process(car(env, '第 %d 辆车' % (i + 1), bcs, i * 2, 5))

env.run()
```
</details>

资源 `resource` 的 `request()` 方法生成一个事件 `event`， 该事件会等待直至资源可用。之后一直拥有资源，直至资源被释放。

在代码中使用 `with` 构造资源的请求使用，资源在使用后将`自动释放`。如果没通过 `with` 使用 `request()`，需要自行使用 `release()` 释放资源。

释放资源后，下一个等待的进程将激活，占用资源。资源的排队规则遵循`先进先出 FIFO`策略。

<details>
<summary>查看结果</summary>

```bash
第 1 辆车 到达时间 0
第 1 辆车 充电开始时间 0
第 2 辆车 到达时间 2
第 2 辆车 充电开始时间 2
第 3 辆车 到达时间 4
第 1 辆车 充电结束并驶离时间 5
第 3 辆车 充电开始时间 5
第 4 辆车 到达时间 6
第 2 辆车 充电结束并驶离时间 7
第 4 辆车 充电开始时间 7
第 3 辆车 充电结束并驶离时间 10
第 4 辆车 充电结束并驶离时间 12
```
</details>

### Example 2.5
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
</details>

<details>
<summary>查看结果</summary>

```bash
Bank renege
 0.0000 Customer00: Here I am
 0.0000 Customer00: Waited  0.000
 3.8595 Customer00: Finished
10.2006 Customer01: Here I am
10.2006 Customer01: Waited  0.000
12.7265 Customer02: Here I am
13.9003 Customer02: RENEGED after  1.174
23.7507 Customer01: Finished
34.9993 Customer03: Here I am
34.9993 Customer03: Waited  0.000
37.9599 Customer03: Finished
40.4798 Customer04: Here I am
40.4798 Customer04: Waited  0.000
43.1401 Customer04: Finished
```
</details>



<br>
<br>
<br>

SimPy 学习  

* {% post_link simpy-learning-01 %}  

* {% post_link simpy-learning-02 %}  

* {% post_link simpy-learning-03 %}