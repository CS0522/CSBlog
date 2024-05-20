---
title: 【学习笔记】SimPy 学习（二）：进程交互
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

SimPy 的 Environment、Events、进程交互等

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

* 可能发生（未触发，不在 event 队列中）
* 会发生（触发，在时间 t 调度并插入 event 队列中）
* 已经发生（处理，从 event 队列中移除）

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