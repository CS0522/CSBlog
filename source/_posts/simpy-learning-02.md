---
title: 【学习笔记】SimPy 学习（二）：核心概念
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

学习 SimPy 的一些核心概念，分为 Environment、Events、进程交互和资源共享

<!-- more -->

## Environment

决定仿真的起点和终点，管理仿真元素之间的关联。

### APIs

* 添加仿真进程：`simpy.Environment.process()`

* 创建事件： `simpy.Environment.event()`

* 提供延时事件：`simpy.Environment.timeout()`

* 仿真结束的条件：`until=xxx`
  * `进程也是事件`：
    ```bash
    proc = env.process(my_proc(env))
    env.run(until=proc)
    ```

* 仿真启动：`simpy.Environment.run()`

* 现在的时间：`simpy.Environment.now`

* 逐个执行仿真事件：
  * 返回下一个计划事件的时间：`simpy.Environment.peek()`
  * 处理下一个计划的事件：`simpy.Environment.step()`
  
  ```bash
  until = 10
  while env.peek() < until:
    env.step()
  ```

* 当前活动进程：`simpy.Environment.active_process`


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

`理解整个进程交互执行的流程。`

<details>
<summary>描述一个汽车驾驶一段时间后停车充电， 汽车驾驶进程和电池充电进程通过事件的激活来相互影响</summary>

```python
import simpy

from random import seed, randint
seed(23)

class ENV:
    def __init__(self, env):
        self.env = env
        self.drive_proc = env.process(self.drive(env))
        self.bat_ctrl_proc = env.process(self.bat_ctrl(env))
        print("执行了 2 个 process")
        self.bat_ctrl_reactivate = env.event()
        self.bat_ctrl_sleep = env.event()
        print("创建了 2 个事件")

    # 驾驶进程
    def drive(self, env):
        while True:
            # drive 20~40 minutes
            print("Start driving at: ", env.now)
            print("drive() 抛出了 timeout")
            yield env.timeout(randint(20, 40))
            print("drive() 恢复了")
            print("End driving at: ", env.now)

            # parking 1~6 hours
            print("Start parking at: ", env.now)
            # activate battery charging
            # 这段代码应该是指触发事件并成功
            print("drive() 触发了 reactivate 并成功")
            self.bat_ctrl_reactivate.succeed()
            self.bat_ctrl_reactivate =  env.event()
            # 继续执行
            # 超时并挂起，等待 bat_ctrl_sleep 事件发生
            print("drive() 抛出了 timeout 和 sleep")
            yield env.timeout(randint(60, 360)) & self.bat_ctrl_sleep
            print("drive() 恢复了")
            print("End parking at: ", env.now)

    # 电池充电进程
    def bat_ctrl(self, env):
        while True:
            print("Charge sleep at: ", env.now)
            # 挂起。接收到 drive() 中的事件发生后恢复
            print("bat_ctrl() 抛出了 reactivate")
            yield self.bat_ctrl_reactivate
            print("bat_ctrl() 恢复了")
            print("Charge activate at: ", env.now)
            print("bat_ctrl() 抛出了 timeout")
            yield env.timeout(randint(30, 90))
            print("bat_ctrl() 恢复了")
            print("Charge end at:", env.now)
            # 这段代码应该是指触发事件并成功
            print("bat_ctrl() 触发了 sleep 并成功")
            self.bat_ctrl_sleep.succeed()
            self.bat_ctrl_sleep = env.event()


if __name__ == "__main__":
    env = simpy.Environment()
    # 下面这一步执行了 init()
    ENV(env)
    print("env 开始模拟...")
    env.run(until=300)
```
</details>

汽车的进程需要引用环境（`env`）来创建新的事件。汽车的行为被描述成一个无限循环。记住，这个函数是一个生成器。尽管它永远不会终止，但一旦到达 `yield` 语句，它会将控制流传回仿真程序。触发生成的事件后（`事件发生`），模拟将在此语句中恢复该函数。

可以看到，首先进行了 `env` 的初始化，进程并没有执行；`env.run()` 后，进程开始执行，个人感觉在 env 环境内进程是同时的，代码是按加入 process 的顺序执行的。

<details>
<summary>查看运行结果</summary>

```bash
执行了 2 个 process
创建了 2 个事件
env 开始模拟...
Start driving at:  0
drive() 抛出了 timeout
Charge sleep at:  0
bat_ctrl() 抛出了 reactivate
drive() 恢复了
End driving at:  29
Start parking at:  29
drive() 触发了 reactivate 并成功
drive() 抛出了 timeout 和 sleep
bat_ctrl() 恢复了
Charge activate at:  29
bat_ctrl() 抛出了 timeout
bat_ctrl() 恢复了
Charge end at: 60
bat_ctrl() 触发了 sleep 并成功
Charge sleep at:  60
bat_ctrl() 抛出了 reactivate
drive() 恢复了
End parking at:  131
Start driving at:  131
drive() 抛出了 timeout
drive() 恢复了
End driving at:  169
Start parking at:  169
drive() 触发了 reactivate 并成功
drive() 抛出了 timeout 和 sleep
bat_ctrl() 恢复了
Charge activate at:  169
bat_ctrl() 抛出了 timeout
bat_ctrl() 恢复了
Charge end at: 226
bat_ctrl() 触发了 sleep 并成功
Charge sleep at:  226
bat_ctrl() 抛出了 reactivate
```
</details>

### Example 2.3

`进程的中断。`

<details>
<summary>在电动汽车还在充电过程中，接到一个紧急通知，需要中断充电进程马上出门</summary>

```python
from random import seed, randint
 
import simpy
seed(23)
 
class EV:
    def __init__(self, env):
        self.env = env
        self.drive_proc = env.process(self.drive(env))
 
 
    def drive(self, env):
        while True:
            # 行驶20-40分钟
            yield env.timeout(randint(20, 40))
 
            # 停车1小时
            print('开始停车时间', env.now)
            charging = env.process(self.bat_ctrl(env))
            print("设置了 timeout")
            parking = env.timeout(60)
            print("抛出了事件")
            yield charging | parking
            if not charging.triggered:
                # 中断充电
                charging.interrupt('该出发了！')
            print('结束停车时间：', env.now)
            
              
    def bat_ctrl(self, env):
        print('充电器启动时间：', env.now)
        try:
            yield env.timeout(randint(60, 90))
            print('充电器完成时间：', env.now)
        except simpy.Interrupt as i:
            # 充电被中断
            print('充电器中断时间：', env.now, ', msg:', i.cause)        
        
env = simpy.Environment()
ev = EV(env)
env.run(until=100)
```
</details>

中断另一个进程。可以对进程调用 `process.interrupt(xxx)`。这将在该进程中引发中断异常，并立即恢复。

`simpy.Interrupt.cause` 获取 `xxx` 信息。

<details>
<summary>查看结果</summary>

```bash
开始停车时间 29
设置了 timeout
抛出了事件
充电器启动时间： 29
结束停车时间： 89
充电器中断时间： 89 , msg: 该出发了！
```
</details>


## Events

SimPy 包含一组用于各种目的的事件类型。它们都继承 `simpy.events.Event`。

事件的层次结构：

* events.Event
  * events.Timeout
  * events.Initialize
  * events.Process
  * events.Condition
    * events.AllOf
    * events.AnyOf


### APIs

* `Event.trigged`
* `Event.processed`
* 触发事件并成功：`event.succeed(value=)`
* 触发事件并失败：`event.fail(exception)`
* `event.trigger(event)`


### 事件基础知识

事件可以处于以下状态之一：

* 可能发生（未触发）
* 会发生（触发）
* 已经发生（处理）

它们按这个顺序经历这些状态一次。`时间`推动事件的状态改变。

* 最初事件没有被触发，它只是内存中的对象

* 如果事件被触发，它将在给定的时间被调度并插入 SimPy 的事件队列中。属性 `Event.trigged` 变为 `True`
  * 只要不处理事件，就可以向事件添加回调。回调接受事件作为参数，储在 event.Callbacks 列表中

* 当 SimPy 将事件从事件队列中弹出并调用其所有回调时，该事件将被处理。现在已无法添加回调。属性 `Event.processed` 变为 `True`。

事件也有值。可以在事件触发之前或触发时设置该值，并可以通过 `event.value` 检索该值，或者在进程中通过生成事件（value=yield event）进行检索。

`进程也是事件`

### 触发事件

当事件被触发时，要么成功要么失败。

* 要触发事件并将其标记为成功，可以使用 `event.succeed(value=None)`，可以选择向它传递一个值（例如，计算结果）

* 要触发事件并将其标记为失败，调用 `event.fail(exception)` 并将异常实例传递给它（例如，在计算失败期间捕获的异常）

* 触发事件的通用方法：`event.trigger(event)`。这将获取传递给它的事件的值和结果（成功或失败）。

这三个方法都返回它们绑定到的事件实例。这允许执行如 `Event(env).succeed()` 之类的操作。

### 同时等待多个事件

SimPy 提供 AnyOf 和 AllOf 事件，这两个事件都是条件事件。两者都将事件列表作为参数，并在触发任何（至少一个）或所有事件时触发。

作为 AllOf 和 AnyOf 的简写，可以使用逻辑运算符 &（and）和 |（or）。

```python
def test_condition(env):
    t1, t2 = env.timeout(1, value='spam'), env.timeout(2, value='eggs')
    ret = yield t1 | t2
    assert ret == {t1: 'spam'}
 
    t1, t2 = env.timeout(1, value='spam'), env.timeout(2, value='eggs')
    ret = yield t1 & t2
    assert ret == {t1: 'spam', t2: 'eggs'}
 
    # You can also concatenate & and |
    e1, e2, e3 = [env.timeout(i) for i in range(3)]
    yield (e1 | e2) & e3
    assert all(e.processed for e in [e1, e2, e3])
 
proc = env.process(test_condition(env))
env.run()
```


## 资源共享

仿真中涉及的人力资源以及工艺上的物料消耗都会抽象用 `Resource` 来表达，主要的 `method` 是 `request`。`Store` 处理各种优先级的队列问题，表现跟 `queue` 一致，通过 `get()/put()` 存放 `item`。

SimPy 定义了三类资源：

* `Resources`：一次可由有限数量的过程使用的资源（例如，具有有限数量燃油泵的加油站）

* `Containers`：模拟同质、未分散的生产、消费资源。它可以是连续的（如水）或离散的（如苹果）

* `Stores`：允许生产和使用 Python 对象的资源

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

### Example 2.6

`PriorityResource` 优先资源。此资源子类允许请求进程为每个请求提供优先级。更重要的请求将比不重要的请求更早地访问资源。优先级用整数表示；`较小的数字意味着较高的优先级`。

<details>
<summary>优先资源例子</summary>

```python
import simpy

def resource_user(name, env, resource, wait, prio):
    print(f'{name} 在 {env.now} 到达，优先级是 {prio}')
    print(f'{name} 挂起...')
    yield env.timeout(wait)
    with resource.request(priority=prio) as req:
        print(f'{name} requesting at {env.now} with priority={prio}')
        yield req
        print(f'{name} got resource at {env.now}')
        yield env.timeout(3)
 
env = simpy.Environment()
res = simpy.PriorityResource(env, capacity=1)
p2 = env.process(resource_user(2, env, res, wait=1, prio=0))
p1 = env.process(resource_user(1, env, res, wait=0, prio=0))
p3 = env.process(resource_user(3, env, res, wait=2, prio=-1))
env.run()
```
</details>

<details>
<summary>查看结果</summary>

```python
2 在 0 到达，优先级是 0
2 挂起...
1 在 0 到达，优先级是 0
1 挂起...
3 在 0 到达，优先级是 -1
3 挂起...
1 requesting at 0 with priority=0
1 got resource at 0
2 requesting at 1 with priority=0
3 requesting at 2 with priority=-1
3 got resource at 3
2 got resource at 6
```
</details>

### Example 2.7

`PreemptiveResource` 抢占资源。

<details>
<summary>抢占资源例子</summary>

```python
import simpy

def resource_user(name, env, resource, wait, prio):
    print(f'{name} 在 {env.now} 到达，优先级是 {prio}')
    print(f'{name} 挂起...')
    yield env.timeout(wait)
    with resource.request(priority=prio) as req:
        print(f'{name} requesting at {env.now} with priority={prio}')
        yield req
        print(f'{name} got resource at {env.now}')
        try:
            yield env.timeout(3)
        except simpy.Interrupt as interrupt:
            by = interrupt.cause.by
            usage = env.now - interrupt.cause.usage_since
            print(f'{name} got preempted by {by} at {env.now}'
                f' after {usage}')
 
env = simpy.Environment()
res = simpy.PreemptiveResource(env, capacity=1)

p3 = env.process(resource_user(3, env, res, wait=2, prio=-1))
p1 = env.process(resource_user(1, env, res, wait=0, prio=0))
p2 = env.process(resource_user(2, env, res, wait=1, prio=0))
env.run()
```
</details>

<details>
<summary>查看结果</summary>

```bash
3 在 0 到达，优先级是 -1
3 挂起...
1 在 0 到达，优先级是 0
1 挂起...
2 在 0 到达，优先级是 0
2 挂起...
1 requesting at 0 with priority=0
1 got resource at 0
2 requesting at 1 with priority=0
3 requesting at 2 with priority=-1
1 got preempted by <Process(resource_user) object at 0x7fa733116898> at 2 after 2
3 got resource at 2
2 got resource at 5
```
</details>

### Example 2.8

抢占请求不允许跳过优先级更高的请求。以下示例显示，在队列中，抢占式的低优先级请求无法插队非抢占式的高优先级请求。

<details>
<summary>抢占式低优先级不允许跳过例子</summary>

```python
import simpy

def user(name, env, res, prio, preempt):
    with res.request(priority=prio, preempt=preempt) as req:
        try:
            print(f'{name} requesting at {env.now}')
            assert isinstance(env.now, int), type(env.now)
            yield req
            assert isinstance(env.now, int), type(env.now)
            print(f'{name} got resource at {env.now}')
            yield env.timeout(3)
        except simpy.Interrupt:
            print(f'{name} got preempted at {env.now}')
 
env = simpy.Environment()
res = simpy.PreemptiveResource(env, capacity=1)
A = env.process(user('A', env, res, prio=0, preempt=True))
env.run(until=1)  # Give A a head start

B = env.process(user('B', env, res, prio=-2, preempt=False))
C = env.process(user('C', env, res, prio=-1, preempt=True))
env.run()
```
</details>

* 处理 A 请求优先级为 0 的资源。它立即成为用户

* 进程 B 请求优先级为 -2 的资源，但将 preempt 设置为 False。它会排队等候

* 进程 C 请求优先级为 -1 的资源，但保留 preempt 为 True。通常情况下，它会抢占 A，但在这种情况下，B 在 C 之前排队，并阻止 C 抢占 A。由于 B 的优先级不够高，C 也不能抢占 B

由于进程B的优先级较高，在本例中不发生抢占。优先级为 -3 的请求可以抢占 A。

<details>
<summary>查看结果</summary>

```bash
A requesting at 0
A got resource at 0
B requesting at 1
C requesting at 1
B got resource at 3
C got resource at 6
```
</details>



<br>
<br>
<br>

SimPy 学习  

* {% post_link simpy-learning-01 %}  

* {% post_link simpy-learning-02 %}  

* {% post_link simpy-learning-03 %}