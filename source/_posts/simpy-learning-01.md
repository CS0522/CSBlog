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

### SimPy 安装
```bash
pip install simpy
```

### SimPy 核心概念

SimPy 是离散事件驱动的仿真库。所有活动部件，例如车辆、顾客，即便是信息，都可以用 `process` 来模拟。这些 `process` 存放在 `environment` 中。所有 `process` 之间，以及与 `environment` 之间的互动，通过 `event` 来进行；`process` 表达为 `generators`，构建 `event` 并通过 `yield` 语句抛出事件；当一个进程抛出事件，进程会被暂停，直到事件被 `triggered`。多个进程可以等待同一个事件。 SimPy 会按照这些进程抛出的事件 `triggered` 的先后， 来恢复进程。

其中最重要的一类事件是 `Timeout`，这类事件允许一段时间后再被激活，用来表达一个进程休眠或者保持当前的状态持续指定的一段时间。这类事件通过 `Environment.timeout` 来调用。

个人理解：
* `process` = 函数
* `environment` = 类（对象）
* `run` = 开始执行函数
* `event` = if/else 语句
* `timeout` = 循环
* `until` = 循环结束条件

#### Environment
决定仿真的起点和终点，管理仿真元素之间的关联

* 添加仿真进程：`simpy.Environment.process`
* 创建事件： `simpy.Environment.event`
* 提供延时事件：`simpy.Environment.timeout`
* 仿真结束的条件：`simpy.Environment.until`
* 仿真启动：`simpyt.Environment.run`

#### Example 1
定义一个进程，仿真的简单代码结构

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