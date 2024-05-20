---
title: 【学习笔记】SimPy 学习（四）：实时模拟与时间调度
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
date: 2023-10-28 22:41:08
---

Simpy 的 Real-time simulations，即设置模拟的时间步长

<!-- more -->

## Real-time-simulation

不以快的速度执行模拟，与<font color="red">现实的时钟时间同步</font>（他是真的执行了这么长时间。如设置 1 分钟，执行 1 分钟）。

* hardware-in-the-loop
* 模拟中存在交互
* 分析算法的实时行为

### APIs

* `simpy.rt.RealtimeEnvironment(initial_time=0, factor=1.0, strict=True)`
  * `factor` 定义模拟时间步长。设置为 0.1，模拟时间单位 1/10 秒；设置为 60，模拟时间单位 1 分钟
  * `strict` 严格判断模拟时间步长内的计算所花费的时间是否超过 `factor` 所允许的时间。如果超过了发生中断

### Example 4.1 

<details>
<summary>简单实时模拟的例子</summary>

```python
import time
import simpy

def example(env):
    start = time.perf_counter()
    # 时延 1 个单位，也就是 1 个 factor
    yield env.timeout(1)
    end = time.perf_counter()
    print('Duration of one simulation time unit: %.2fs' % (end - start))

env = simpy.Environment()
proc = env.process(example(env))
env.run(until=proc)

import simpy.rt
env = simpy.rt.RealtimeEnvironment(factor=10)
proc = env.process(example(env))
env.run(until=proc)
```
</details>

<details>
<summary>查看结果</summary>

```bash
Duration of one simulation time unit: 0.00s
Duration of one simulation time unit: 10.01s
``` 
</details>


## Time and Scheduling

SimPy 是一个单线程、确定性的库，按顺序逐个处理事件。如果同时安排两个事件，则首先安排的事件也将是最先处理的事件（FIFO）。

模拟环境中的进程是<font color="red">并行</font>运行，但当模拟在 CPU 上运行时，所有事件会按顺序进行确定处理。如果多次运行模拟（不使用 `random` 情况下），则总会得到相同的结果。

SimPy 的事件队列被实现为堆队列。因此，将事件作为元组 `(t, event)`（t 是计划时间）插入其中，那么队列中的第一个元素将始终是t最小的元素，下一个元素将被处理。

如果同时调度两个事件，则存储 `(t, event)` 元组将不起作用。为了解决这个问题，SimPy 还存储了一个严格递增的事件 id: `(t, eid, event)`。这样，如果两个事件被安排在同一时间，那么首先被安排的事件（id 更小）将始终被首先处理。




<br>
<br>
<br>

SimPy 学习  

* {% post_link n-simpy-learning-01 %}  

* {% post_link n-simpy-learning-02 %}  

* {% post_link n-simpy-learning-03 %}

* {% post_link n-simpy-learning-04 %}

* {% post_link n-simpy-learning-05 %}

* {% post_link n-simpy-learning-06 %}