---
title: 【学习笔记】SimPy 学习（五）：SimPy 版本迁移
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
date: 2023-10-29 22:27:58
---

SimPy 版本迁移：SimPy 2 -> SimPy 3 -> SimPy 4

<!-- more -->

> [API Reference](https://simpy.readthedocs.io/en/latest/api_reference/index.html)

## SimPy 2 to 3

### Imports

SimPy 3 的 `import` 被简化。SimPy 2 需要根据模拟环境选择需要导入的模块，SimPy 3 大部分情况下只用导入 `simpy`

```python
# SimPy 2
from Simpy.Simulation import Simulation, Process, hold
```
```python
# SimPy 3
import simpy
```

### Simulation* classes

SimPy 3 的 `Environment` 替换 `Simulation`，`RealtimeEnvironment` 替换 `SimulationRT`。

```python
# SimPy 2
# Procedural API
from SimPy.Simulation import initialize, simulate

initialize()
# Start processes
simulate(until=10)

# Object-oriented API
from SimPy.Simulation import Simulation

sim = Simulation()
# Start processes
sim.simulate(until=10)
```
```python
# SimPy 3
import simpy

env = simpy.Environment()
# Start processes
env.run(until=10)
```

### Defining a Process

SimPy 3 中的 Process 是一个封装在进程实例中的 Python 生成器（无论它是在模块级定义的还是作为实例方法定义的）。生成器通常需要对环境的引用才能与之交互，但这是可选的。

进程可以通过创建一个进程实例并将生成器传递给它来启动。环境为此提供了一个快捷方式：`process()`。

```python
# SimPy 2
# Procedural API
from Simpy.Simulation import Process

class MyProcess(Process):
    def __init__(self, another_param):
        super().__init__()
        self.another_param = another_param

    def generator_function(self):
        """Implement the process' behavior."""
        yield something

initialize()
proc = Process('Spam')
activate(proc, proc.generator_function())

# Object-oriented API
from SimPy.Simulation import Simulation, Process

class MyProcess(Process):
    def __init__(self, sim, another_param):
        super().__init__(sim=sim)
        self.another_param = another_param

    def generator_function(self):
        """Implement the process' behaviour."""
        yield something

sim = Simulation()
proc = Process(sim, 'Spam')
sim.activate(proc, proc.generator_function())
```

```python
# SimPy 3
import simpy

def generator_function(env, another_param):
    """Implement the process' behavior."""
    yield something

env = simpy.Environment()
proc = env.process(generator_function(env, 'Spam'))
```

### SimPy Keywords

SimPy 3中，可以直接生成事件。可以直接实例化事件，也可以使用 Environment 提供的快捷方式。

通常，每当一个进程产生一个事件时，该进程的执行都会被挂起，并在事件被触发后重新开始。为了激发这种理解，一些活动被重新命名。例如，`hold` 关键字意味着等待一段时间。就事件而言，这意味着发生了超时。因此，`hold` 已被 `Timeout` 事件取代。

```python
# SimPy 2
yield hold, self, duration
yield passivate, self
yield request, self, resource
yield release, self, resource
yield waitevent, self, event
yield waitevent, self, [event_a, event_b, event_c]
yield queueevent, self, event_list
yield get, self, level, amount
yield put, self, level, amount
```

```python
# SimPy 3
yield env.timeout(duration)        # hold: renamed
yield env.event()                  # passivate: renamed
yield resource.request()           # Request is now bound to class Resource
resource.release()                 # Release no longer needs to be yielded
yield event                        # waitevent: just yield the event
yield env.all_of([event_a, event_b, event_c])  # waitevent
yield env.any_of([event_a, event_b, event_c])  # queuevent
yield container.get(amount)        # Level is now called Container
yield container.put(amount)

yield event_a | event_b            # Wait for either a or b. This is new.
yield event_a & event_b            # Wait for a and b. This is new.
yield env.process(calculation(env))  # Wait for the process calculation to
                                     # to finish.
```

### Interrupts

SimPy 3 中，可以对进程调用 `interrupt()`。一个中断被抛出到进程中，进程必须通过 `try...except simpy.Interrupt` 处理中断。

```python
# SimPy 2
class Interrupter(Process):
    def __init__(self, victim):
        super().__init__()
        self.victim = victim

    def run(self):
        yield hold, self, 1
        self.interrupt(self.victim_proc)
        self.victim_proc.interruptCause = 'Spam'

class Victim(Process):
    def run(self):
        yield hold, self, 10
        if self.interrupted:
            cause = self.interruptCause
            self.interruptReset()
```

```python
# SimPy 3
def interrupter(env, victim_proc):
    yield env.timeout(1)
    victim_proc.interrupt('Spam')

def victim(env):
    try:
        yield env.timeout(10)
    except Interrupt as interrupt:
        cause = interrupt.cause
```


## SimPy 3 to 4

Python >= 3.6

### Environment Subclasses

`BaseEnvironment` 类已在 SimPy 4 中移除。`Environment` 类现在是最基本的类。任何从 `BaseEnvironment` 继承的代码都应该修改为从 `Environment` 继承。

```python
# SimPy 3
class MyEnv(SimPy.BaseEnvironment):
  ...

# SimPy 4
class MyEnv(SimPy.Environment):
  ...
```

### Returning from Process Generators

在下面的示例中，`Environment.exit()` 用于返回 the first needle。当将 SimPy 3 与 Python 2 一起使用时，这是从流程生成器返回值的唯一方法。

```python
# SimPy 3 + Python 2 
def find_first_needle(env, store):
    while True:
        item = yield store.get()
        if is_needle(item):
            env.exit(item)  # Python2 generators cannot use return

def proc(env, store):
    needle = yield env.process(find_first_needle(env, store))
```

在 SimPy 4 与 Python 3 下，可以写为：

```python
def find_first_needle(env, store):
    while True:
        item = yield store.get()
        if is_needle(item):
            return item  # A Python3 generator can return
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