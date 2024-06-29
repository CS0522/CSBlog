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

Python 仿真库 SimPy，安装、结构和基本概念

<!-- more -->

> 学习笔记参考[SimPy 文档](https://simpy.readthedocs.io/en/latest/)   

## SimPy 简介

> "SimPy 是基于过程的`离散事件`的标准 Python 模拟框架"  
> [SimPy 文档](https://simpy.readthedocs.io/en/latest/)  

## SimPy 安装
```bash
pip install simpy
```

## SimPy 基本概念

SimPy 是`离散事件`驱动的仿真库。所有活动部件，例如车辆、顾客，即便是信息，都可以用 `process` 来模拟。这些 `process` 存放在 `environment` 中。所有 `process` 之间，以及与 `environment` 之间的互动，通过 `event` 来进行。

SimPy 可以看作一个异步事件分发器，生成一些事件并在给定的仿真时间安排它们。事件按`优先级`、`仿真时间`和`递增的事件id`排序。事件还具有回调列表，这些回调在事件触发和处理时执行。事件也可能有返回值。

涉及的组件是编写的`环境（Envirement）`、`事件（events）`和`进程（process）函数`。

### 环境（Environment）
`环境（Environment）`将这些`事件（events）`存储在其事件列表中，并跟踪当前仿真时间

### 进程（process）
`进程（process）函数`实现仿真模型，即定义仿真行为。它们可以看作是挂起事件实例的普通的 Python 生成器函数。进程在生命周期中可以创造并`挂起（yield）事件（Events）`，等待事件的触发

* `当进程生成事件时，该进程将被挂起`。当事件发生时（事件已触发），SimPy 恢复进程。多个进程可以等待同一事件。SimPy 按照生成事件的统一顺序进行恢复
  
* 启动`进程`涉及两步：
  1. 必须调用 Process 函数才能创建生成器对象。（这不会执行该函数的任何代码。见 [Python yield 关键字](#python-yield-关键字)）
   
  2. 然后创建进程实例，并将环境和生成器对象传递给它。这将在当前仿真时间安排一个初始化事件，启动进程函数的执行。进程实例也是进程函数返回时触发的事件。

* 默认情况下，只要事件列表中有事件，它就会运行，但您也可以通过提供until参数让它提前停止（请参见仿真控件）。

### 事件（events）
其中一类重要的`事件`是 `Timeout`，这类事件允许一段时间后再被激活，用来表达一个进程休眠或者保持当前的状态持续指定的一段时间。这类事件通过 `Environment.timeout` 来调用



## Python yield 关键字

看作 `return`。举例说明。

### Example 1.1

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


### Example 1.2

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


## Python 生成器

使用了 `yield` 的函数被称为`生成器函数`。生成器函数可以在迭代过程中逐步产生值，而不是一次性返回所有结果。生成器是一个返回迭代器的函数，只能用于迭代操作，更简单点理解生成器就是一个迭代器。

当在生成器函数中使用 `yield` 语句时，函数的执行将会暂停，并将 `yield` 后面的表达式作为当前迭代的值返回；然后，每次调用生成器的 `next()` 方法或使用 for 循环进行迭代时，函数会从上次暂停的地方继续执行，直到再次遇到 `yield` 语句。这样，生成器函数可以逐步产生值，而不需要一次性计算并返回所有结果。

生成器函数的优势是它们可以按需生成值，避免一次性生成大量数据并占用大量内存。

调用一个生成器函数，返回的是一个`迭代器对象`。

### Example 1.3

生成器函数实现费波拉契数列

```python
import sys
 
def fibonacci(n): # 生成器函数
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n): 
            return
        yield a
        a, b = b, a + b
        counter += 1

f = fibonacci(10) # f 是一个迭代器，由生成器返回生成
 
while True:
    try:
        # end=" " 意思是每输出一个数，以" "结尾，而不是换行
        print (next(f), end=" ")
    except StopIteration:
        sys.exit()
```

运行结果

```bash
0 1 1 2 3 5 8 13 21 34 55
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