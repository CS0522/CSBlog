---
title: 【学习笔记】SimPy 学习（三）：资源共享
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
date: 2023-10-27 16:24:09
---

SimPy 的资源共享，Resource、Container、Store 等

<!-- more -->

仿真中涉及的人力资源以及工艺上的物料消耗都会抽象用 `Resource` 来表达，主要的 `method` 是 `request`。`Store` 处理各种优先级的队列问题，表现跟 `queue` 一致，通过 `get()/put()` 存放 `item`。

```python
BaseResource(capacity):
  put_queue
  get_queue

  put(): event
  get(): event
```

每个资源都有一个最大容量和两个队列：一个队列用于要向其中放入内容的进程，另一个队列则用于要取出内容的进程。`put()` 和 `get()` 方法都返回一个在相应操作成功时触发的事件。

SimPy 定义了三类资源：

* `Resources`：一次可由有限数量的过程使用的资源（例如，具有有限数量燃油泵的加油站）
  * 资源或限制条件：`simpy.Resource(env, capacity)`
  * 优先级、不可打断正在服务的进程：`simpy.PriorityResource`
  * 优先级、可以打断正在服务的进程：`simpy.PreemptiveResource`

* `Containers`：模拟同质、未分散的生产、消费资源。它可以是连续的（如水）或离散的（如苹果）
  * 连续不可分的元素：`simpy.Container`

* `Stores`：允许生产和使用 Python 对象的资源
  * 存取 `Item`、遵循先来后到：`simpy.Store`
  * 添加优先级：`simpy.PriorityStore`
  * 存在分类：`simpy.FilterStore`


## Resources

### APIs

* `simpy.Resource(env, capacity = )`
* `simpy.Resource.request(priority = )`
* `simpy.Resource.release()`

### Example 3.1
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

### Example 3.2
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

### Example 3.3

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

### Example 3.4

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

### Example 3.5

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


## Containers

### APIs

* `simpy.Container(env, init = , capacity = )`
* `simpy.Container.level`
* `simpy.Container.put()`
* `simpy.Container.get()`

### Example 3.6

<details>
<summary>模拟加油站的 tank。Tanker 增加了 tank 中的汽油量，而 Car 则减少了汽油量</summary>

```python
import simpy

class GasStation:
    def __init__(self, env):
        # 2 个加油机
        self.fuel_dispensers = simpy.Resource(env, capacity=2)
        # tank 容量为 1000，初始 100
        self.gas_tank = simpy.Container(env, init=100, capacity=1000)
        self.mon_proc = env.process(self.monitor_tank(env))

    # 监视是否需要加油
    def monitor_tank(self, env):
        while True:
            if self.gas_tank.level < 100:
                print(f'Calling tanker at {env.now}')
                env.process(tanker(env, self))
            yield env.timeout(15)


def tanker(env, gas_station):
    yield env.timeout(10)  # Need 10 Minutes to arrive
    print(f'Tanker arriving at {env.now}')
    # tank 加满
    amount = gas_station.gas_tank.capacity - gas_station.gas_tank.level
    yield gas_station.gas_tank.put(amount)


def car(name, env, gas_station):
    print(f'Car {name} arriving at {env.now}')
    with gas_station.fuel_dispensers.request() as req:
        yield req
        print(f'Car {name} starts refueling at {env.now}')
        yield gas_station.gas_tank.get(40)
        yield env.timeout(5)
        print(f'Car {name} done refueling at {env.now}')


def car_generator(env, gas_station):
    for i in range(4):
        env.process(car(i, env, gas_station))
        yield env.timeout(5)


env = simpy.Environment()
gas_station = GasStation(env)
car_gen = env.process(car_generator(env, gas_station))
env.run(35)
```
</details>

加油站具有数量有限的加油机（建模为 `Resource`）和油箱（建模为 `Container`）。可以检查其当前液位及其容量（`GasStation.monitor_tank()` 和 `tanker()`）。可以通过 `put_queue` 和 `get_queue`属性（类似于`Resource.queue`）访问等待事件列表。

<details>
<summary>查看结果</summary>

```bash
Car 0 arriving at 0
Car 0 starts refueling at 0
Car 1 arriving at 5
Car 0 done refueling at 5
Car 1 starts refueling at 5
Car 2 arriving at 10
Car 1 done refueling at 10
Car 2 starts refueling at 10
Calling tanker at 15
Car 3 arriving at 15
Car 3 starts refueling at 15
Tanker arriving at 25
Car 2 done refueling at 30
Car 3 done refueling at 30
```
</details>


## Stores

使用 `Stores` 可以对具体对象的生产和消费进行建模。单个 `Store` 甚至可以包含多种类型的对象。

### APIs

* `simpy.Store(env, capcity = )`
* `simpy.Store.get()`
* `simpy.Store.put()`
* `simpy.PriorityStore()`
* `simpy.PriorityItem(priority, item)`

### Example 3.7

<details>
<summary>一般生产者/消费者场景</summary>

```python
import simpy

def producer(env, store):
    for i in range(100):
        yield env.timeout(2)
        yield store.put(f'spam {i}')
        print(f'Produced spam at', env.now)


def consumer(name, env, store):
    while True:
        yield env.timeout(1)
        print(name, 'requesting spam at', env.now)
        item = yield store.get()
        print(name, 'got', item, 'at', env.now)


env = simpy.Environment()
store = simpy.Store(env, capacity=2)

prod = env.process(producer(env, store))
consumers = [env.process(consumer(i, env, store)) for i in range(2)]

env.run(until=10)
```
</details>

<details>
<summary>查看结果</summary>

```bash
0 requesting spam at 1
1 requesting spam at 1
Produced spam at 2
0 got spam 0 at 2
0 requesting spam at 3
Produced spam at 4
1 got spam 1 at 4
1 requesting spam at 5
Produced spam at 6
0 got spam 2 at 6
0 requesting spam at 7
Produced spam at 8
1 got spam 3 at 8
1 requesting spam at 9
```
</details>

### Example 3.8

`PriorityStore` 具有优先级的 Store。

<details>
<summary>inspector process 发现并记录单独的 maintainer process 按优先级顺序修复的问题。</summary>

```python
import simpy

env = simpy.Environment()
issues = simpy.PriorityStore(env)

def inspector(env, issues):
    for issue in [simpy.PriorityItem('P2', '#0000'),
                  simpy.PriorityItem('P0', '#0001'),
                  simpy.PriorityItem('P3', '#0002'),
                  simpy.PriorityItem('P1', '#0003')]:
        yield env.timeout(1)
        print(env.now, 'log', issue)
        yield issues.put(issue)

def maintainer(env, issues):
    while True:
        yield env.timeout(3)
        issue = yield issues.get()
        print(env.now, 'repair', issue)

_ = env.process(inspector(env, issues))
_ = env.process(maintainer(env, issues))
env.run()
```
</details>


<details>
<summary>查看结果</summary>

```bash
1 log PriorityItem(priority='P2', item='#0000')
2 log PriorityItem(priority='P0', item='#0001')
3 log PriorityItem(priority='P3', item='#0002')
3 repair PriorityItem(priority='P0', item='#0001')
4 log PriorityItem(priority='P1', item='#0003')
6 repair PriorityItem(priority='P1', item='#0003')
9 repair PriorityItem(priority='P2', item='#0000')
12 repair PriorityItem(priority='P3', item='#0002')
```
</details>



<br>
<br>
<br>

SimPy 学习  

* {% post_link simpy-learning-01 %}  

* {% post_link simpy-learning-02 %}  

* {% post_link simpy-learning-03 %}

* {% post_link simpy-learning-04 %}

* {% post_link simpy-learning-05 %}