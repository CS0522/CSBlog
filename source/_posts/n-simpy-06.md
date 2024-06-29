---
title: 【学习笔记】SimPy 学习（六）：监视
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
date: 2023-11-02 22:27:58
---

SimPy Monitoring

<!-- more -->

## Monitor processes

在一个或多个状态变量每次更改时或以离散的间隔监视它们的值，并将其存储在某个位置（例如，内存、数据库或文件中）

在以下例子中，只需使用一个列表，并在每次更改时附加所需的值。

```python
import simpy

data = []  # This list will hold all collected data

def test_process(env, data):
    val = 0
    for i in range(5):
        val += env.now
        data.append(val)  # Collect data
        yield env.timeout(1)

env = simpy.Environment()
p = env.process(test_process(env, data))
env.run(p)
print('Collected', data)
```

```bash
Collected [0, 1, 3, 6, 10]
```

如果要监视多个变量，可以将 `(named)tuples` 附加到数据列表中。

如果将数据存储在 NumPy 数组或数据库中，可以通过将数据缓冲在 Python 列表中，并且只向数据库写入较大的块（或完整的数据集）。


## Resource usage

资源的监视有很多种可能的情况，比如：

* 一段时间内的资源利用率，
  
  * 一次使用资源的进程数
  * container 的 level
  * store 中的 item 数量
  
  这可以在离散的时间步长中进行监控，也可以在每次发生变化时进行监控

* 一段时间内（put | get）队列中的进程数（以及平均值）。也可以在离散的时间步长或每次发生变化时进行监控

* 对于 PreemptiveResource，可能需要测量一段时间内发生抢占的频率

没有直接的访问资源的代码，但可以通过 `Monkey-patching` 方法收集。

以下是一个示例，演示了如何向在 `get/request` 或 `put/release` 事件之前或之后被调用的资源添加回调：

```python
from functools import partial, wraps
import simpy

def patch_resource(resource, pre=None, post=None):
    """Patch *resource* so that it calls the callable *pre* before each
    put/get/request/release operation and the callable *post* after each
    operation.  The only argument to these functions is the resource
    instance.

    """
    def get_wrapper(func):
        # Generate a wrapper for put/get/request/release
        @wraps(func)
        def wrapper(*args, **kwargs):
            # This is the actual wrapper
            # Call "pre" callback
            if pre:
                pre(resource)

            # Perform actual operation
            ret = func(*args, **kwargs)

            # Call "post" callback
            if post:
                post(resource)

            return ret
        return wrapper

    # Replace the original operations with our wrapper
    for name in ['put', 'get', 'request', 'release']:
        if hasattr(resource, name):
            setattr(resource, name, get_wrapper(getattr(resource, name)))

def monitor(data, resource):
    """This is our monitoring callback."""
    item = (
        resource._env.now,  # The current simulation time
        resource.count,  # The number of users
        len(resource.queue),  # The number of queued processes
    )
    data.append(item)

def test_process(env, res):
    with res.request() as req:
        yield req
        yield env.timeout(1)

env = simpy.Environment()

res = simpy.Resource(env, capacity=1)
data = []
# Bind *data* as first argument to monitor()
# see https://docs.python.org/3/library/functools.html#functools.partial
monitor = partial(monitor, data)
patch_resource(res, post=monitor)  # Patches (only) this resource instance

p = env.process(test_process(env, res))
env.run(p)

print(data)
```

```bash
[(0, 1, 0), (1, 0, 0)]
```

查看一次有多少个 processes 在等待同一个资源：

```python
import simpy

class MonitoredResource(simpy.Resource):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.data = []

    def request(self, *args, **kwargs):
        self.data.append((self._env.now, len(self.queue)))
        return super().request(*args, **kwargs)

    def release(self, *args, **kwargs):
        self.data.append((self._env.now, len(self.queue)))
        return super().release(*args, **kwargs)

def test_process(env, res):
    with res.request() as req:
        yield req
        yield env.timeout(1)

env = simpy.Environment()

res = MonitoredResource(env, capacity=1)
p1 = env.process(test_process(env, res))
p2 = env.process(test_process(env, res))
env.run()

print(res.data)
```

```bash
[(0, 0), (0, 0), (1, 1), (2, 0)]
```


## Event tracing

跟踪事件的创建、触发和处理时间，哪个进程创建了事件，哪些进程等待了事件。

* `Environment.step()`：处理所有事件
* `Environment.schedule()`：安排所有事件、插入 SimPy 事件队列中

Environment.step() 跟踪所有处理的事件：

```python
from functools import partial, wraps
import simpy

def trace(env, callback):
    """Replace the ``step()`` method of *env* with a tracing function
    that calls *callbacks* with an events time, priority, ID and its
    instance just before it is processed.

    """
    def get_wrapper(env_step, callback):
        """Generate the wrapper for env.step()."""
        @wraps(env_step)
        def tracing_step():
            """Call *callback* for the next event if one exist before
            calling ``env.step()``."""
            if len(env._queue):
                t, prio, eid, event = env._queue[0]
                callback(t, prio, eid, event)
            return env_step()
        return tracing_step

    env.step = get_wrapper(env.step, callback)

def monitor(data, t, prio, eid, event):
    data.append((t, eid, type(event)))

def test_process(env):
    yield env.timeout(1)

data = []
# Bind *data* as first argument to monitor()
# see https://docs.python.org/3/library/functools.html#functools.partial
monitor = partial(monitor, data)

env = simpy.Environment()
trace(env, monitor)

p = env.process(test_process(env))
env.run(until=p)

for d in data:
    print(d)
```

```bash
(0, 0, <class 'simpy.events.Initialize'>)
(1, 1, <class 'simpy.events.Timeout'>)
(1, 2, <class 'simpy.events.Process'>)
```



<br>
<br>
<br>

SimPy 学习  

* {% post_link n-simpy-01 %}  

* {% post_link n-simpy-02 %}  

* {% post_link n-simpy-03 %}

* {% post_link n-simpy-04 %}

* {% post_link n-simpy-05 %}

* {% post_link n-simpy-06 %}