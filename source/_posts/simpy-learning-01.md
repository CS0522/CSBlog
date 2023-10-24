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
  - SimPy
comments: false
cover: false
date: 2023-10-23 21:00:52
---

Python 仿真库 SimPy，基本安装和简单结构

<!-- more -->

> 学习笔记参考[]()  
> 

## SimPy 简介
> "SimPy 是基于过程的`离散事件`的标准 Python 模拟框架"  
> [SimPy 文档](https://simpy.readthedocs.io/en/latest/)  

## SimPy 安装
```bash
pip install simpy
```

## SimPy 基本概念

SimPy 是`离散事件`驱动的仿真库。所有活动部件，例如车辆、顾客，即便是信息，都可以用 `process` 来模拟。这些 `process` 存放在 `environment` 中。所有 `process` 之间，以及与 `environment` 之间的互动，通过 `event` 来进行。

进程由简单的 Python 生成器描述。可以通过 function 或 method 调用它，这取决于它是一个普通函数还是一个类的方法。进程在生命周期中可以创造并挂起（`yield`）事件（`Events`），等待事件的触发。

当进程生成事件时，该进程将被挂起。当事件发生时（事件已触发），SimPy 恢复进程。多个进程可以等待同一事件。SimPy 按照生成事件的统一顺序进行恢复。

其中最重要的一类事件是 `Timeout`，这类事件允许一段时间后再被激活，用来表达一个进程休眠或者保持当前的状态持续指定的一段时间。这类事件通过 `Environment.timeout` 来调用。

个人理解：
* `process` = 函数
* `environment` = 类（对象）
* `run` = 开始执行实例的函数
* `event` = if/else 语句
* `timeout` = 超时事件
* `until` = 循环结束条件
* `yield` = 挂起，并等待事情发生（触发）以恢复进程


## Python `yield` 关键字

当作 `return` 进行思考。举例说明。

### Example 01

```python
def foo():
    print("starting...")
    while True:
        res = yield 4
        print("res:",res)
g = foo()
print(next(g))
print("*"*20)
print(next(g))
```

```bash
starting...
4
********************
res: None
4
```

运行过程：
* 程序开始执行以后，因为 `foo()` 中有 `yield` 关键字，所以 `foo()` 并不会真的执行，而是先得到一个生成器 g（相当于一个对象）；
* 调用 `next(g)`，`foo()` 正式开始执行，先执行 `print("starting...")`，然后进入 `while`；
* 程序遇到 `yield` 关键字，把`yield` 想成 `return`。`return` 了一个 4 之后，程序停止，并没有执行赋值给 res 操作，此时 `next(g)` 执行完成，所以输出的前两行第一个是 while 上面 print 的结果,第二个是return出的结果；
* 程序执行 `print("*"20)`；
* 又开始执行下面的 `print(next(g))`，从刚才那个next程序停止的地方开始执行，也就是要执行 res 的赋值操作，`这个时候赋值操作的右边是没有值的`（因为刚才那个是return出去了，并没有给赋值操作的左边传参数），所以这个时候 res 赋值是 None，所以接着下面的输出就是 `res:None`；
* 程序会继续在 while 里执行，又一次碰到`yield`，这个时候同样 return 出 4，然后程序停止，print 函数输出的 4 就是这次 return 出的 4。


### Example 02

```python
def foo():
    print("starting...")
    while True:
        res = yield 4
        print("res:",res)
g = foo()
print(next(g))
print("*"*20)
###
print(g.send(7))
```

```bash
starting...
4
********************
res: 7
4
```

运行过程：
* 程序执行 `g.send(7)`，程序会从 `yield`关键字那一行继续向下运行，`send` 会把 7 这个值赋值给 res 变量；
* 由于 `send` 方法中包含 `next()` 方法，所以程序会继续向下运行执行 `print`，然后再次进入 while；
* 程序执行再次遇到 `yield` 关键字，`yield` 会返回后面的值后，程序再次暂停，直到再次调用 `next`方法或 `send` 方法。