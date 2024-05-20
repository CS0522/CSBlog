---
title: 【学习笔记】一个用 Python 2 + SimPy 2 实现的 DiskSim
tags:
  - SimPy
  - Python
  - DiskSim
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - SimPy
comments: false
cover: false
date: 2023-11-05 15:46:05
---

学习一个用 Python 2 + SimPy 2 实现的 DiskSim 磁盘模拟器项目

<!-- more -->

项目包含两个脚本：

* `DiskSim.py`：实现模拟器的主要功能
* `ReadQueue.py`：实现 ReadQueue 队列，根据算法调整

项目环境为以下：

* Python 2.7
* SimPy 2.3.1
* numpy 1.7.1
* matplotlib 1.2.1 (no support for Python 3+)

打算按照这个项目进行修改，从 Python 2 + SimPy 2 迁移至 Python 3 + SimPy 4，并添加其他的功能


## FCFS、SSTF、Elevator/SCAN 磁盘调度算法

> 参考[图解五种磁盘调度算法, FCFS, SSTF, SCAN, C-SCAN, LOOK](https://blog.csdn.net/qq_40212930/article/details/105393493)

### FCFS

先来先服务（FCFS）算法。虽然这种算法比较公平，但是它通常并不提供最快的服务。

磁盘队列 I/O 请求块的柱面的顺序：98,183,37,122,14,124,65,67

调度如下图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-disksim-simpy/fig01.png)

从 122 到 14 再到 124 的大摆动说明了这种调度的问题。如果对柱面 37 和 14 的请求一起处理，不管是在 122 和 124 之前或之后，总的磁头移动会大大减少，并且性能也会因此得以改善。


### SSTF 最短寻道优先

SSTF 算法<font color="red">选择处理距离当前磁头位置的最短寻道时间的请求。换句话说，SSTF 选择最接近磁头位置的待处理请求。</font>

对于上面请求队列的示例，与开始磁头位置（53）的最近请求位于柱面 65。一旦位于柱面 65，下个最近请求位于柱面 67。从那里，由于柱面 37 比 98 还要近，所以下次处理 37。如此，会处理位于柱面 14 的请求，接着 98，122，124，最后183（图 2）。这种调度算法的磁头移动只有 236 个柱面。

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-disksim-simpy/fig02.png)

SSTF 调度本质上是一种最短作业优先（SJF）调度；与 SJF 调度一样，它可能会导致一些请求的饥饿。假设在队列中有两个请求，分别针对柱面 14 和 186，而当处理来自 14 的请求时，另一个靠近 14 的请求来了，这个新的请求会下次处理，这样位于 186 的请求需要等待。而当这样的新的请求足够多的话，186 的请求就得不到服务。

SSTF 的算法并非最优。从 53 到 37 再到 14，移动的柱面总数更少，为 208。


### Elevator/SCAN

磁臂从磁盘的一端开始，向另一端移动；在移过每个柱面时，处理请求。当到达磁盘的另一端时，磁头移动方向反转，并继续处理。磁头连续来回扫描磁盘。

如图所示。在采用 SCAN 来调度柱面 98、183、37、122、14、124、65 和 67 的请求之前，除了磁头的当前位置，还需知道磁头的移动方向。

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-disksim-simpy/fig03.png)

假设磁头朝 0 移动并且磁头初始位置还是 53，磁头接下来处理 37，然后 14。在柱面 0 时，磁头会反转，移向磁盘的另一端，并处理柱面 65、67、98、122、124、183（图 3）上的请求。如果请求刚好在磁头前方加入队列，则它几乎马上就会得到服务；如果请求刚好在磁头后方加入队列，则它必须等待，直到磁头移到磁盘的另一端，反转方向，并返回。


### C-SCAN



### LOOK



## DiskSim.py

实现模拟器的主要功能

<details>
<summary>完整代码</summary>

```python
from SimPy.Simulation import *
from random import expovariate
import argparse
import ReadQueue
import numpy as np
import matplotlib.pyplot as plt

MOV_TIME = .4
NUM_ALGS = 3
NUM_TESTS = 4
BYTES_PER_SEC = 256
SECTORS = 128
DISK_REQUESTS = 1000
MEAN_READ_LENGTH = 4
MAXTIME = 1000000.0
        
class Controller(Process):
    """Disk controller generates DISK_REQUESTS # of requests for read head, 
    distributed uniformly with average interarrival time specified by the user."""
    def generate(self, mean):
        for i in range(DISK_REQUESTS):
            req = ReadRequest(name="Request%03d" % (i), sim=self.sim)
            self.sim.activate(req, req.request())
            nextReq = random.uniform(0, mean*2)
            if (i == 100):
                self.sim.byteMon.reset()
                self.sim.seekMon.reset()
                self.sim.accessMon.reset()
                self.sim.rotMon.reset()
                self.sim.diskMov.reset()
            yield hold, self, nextReq
            
class ReadRequest(Process):
    """ReadRequest class simulates a disk head read request sent to the arm"""
    def request(self):
        self.trackNum = int(random.uniform(0, 100))
        readLength = expovariate(1.0/MEAN_READ_LENGTH)
        self.sim.byteMon.observe(readLength*BYTES_PER_SEC)
        yield request,self,self.sim.head,self.trackNum
        trackDis = abs(self.sim.head.pos - self.trackNum)
        self.sim.diskMov.observe(trackDis)
        seekTime = trackDis * MOV_TIME
        self.sim.seekMon.observe(seekTime)
        self.sim.head.pos = self.trackNum
        rotTime = (float(SECTORS)/float(self.sim.head.diskRpm)) * readLength
        self.sim.rotMon.observe(rotTime)
        accessTime = seekTime + rotTime
        self.sim.accessMon.observe(accessTime)
        yield hold,self,accessTime
        yield release,self,self.sim.head

class DiskSim(Simulation):
    """Main Simulation class. Accepts command line input from the user, specifying
    which algorithm to run, the average inter arrival time, and the maximum rpm 
    speed of the disk. Optionally, a test mode can be invoked to test all variants
    and graphically display their results.
    Usage from command line:
    python DiskSim.py -a FCFS -i 25 -r 7000
    ---
    python DiskSim.py -test True
    """
    def __init__(self):
        """Constructor for DiskSim simulation"""
        parser = argparse.ArgumentParser(description='Hard Disk Simulation')
        parser.add_argument('-test',default=False,help="""test mode will run through
        the 12 standard test for the disk simulator.""")
        parser.add_argument('-a', choices=['FCFS','SSF','SCAN'],default='SCAN', 
        help="""the algorithm used by the read head when selecting the next 
        sector. Choices are FCFS, SSF, or SCAN. Default is FCFS""")
        parser.add_argument('-i', default=5, type=int, help="""average inter-arrival
        times of disk requests. Will be uniformly distributed; default is 150""")
        parser.add_argument('-r', default=7200, type=int, help="""maximum rotations 
        per minute achieveable by the hard disk. Default is 7200""")
        args = parser.parse_args()
        if (args.test):
            runTests()
        print("\nAlgorithm: %s"%args.a)
        print("Inter-arrival: %d"%args.i)
        print("Disk RPM: %d"%args.r)
        self.initialize()
        self.accessMon = Monitor('Access time (Seek + Rotation)', sim=self)
        self.seekMon = Monitor('Seek time', sim=self)
        self.rotMon = Monitor('Rotational latency monitor', sim=self)
        self.diskMov = Monitor('Disk arm movement', sim=self)
        self.byteMon = Monitor('Total bytes read', sim=self)
        self.head = Resource(name="ReadHead",qType= ReadQueue.ReadQ, sim=self)
        self.head.algorithm = args.a
        self.head.diskRpm = args.r
        self.head.pos = 0
        controller = Controller('Controller',sim=self)
        self.activate(controller, controller.generate(mean=args.i), at=0.0)
        self.simulate(until=MAXTIME)
        print("Average seek time: %f" %self.seekMon.mean())
        print("Average rotational latency: %F"%self.rotMon.mean())
        print("Average access time: %f"%self.accessMon.mean())
        print("Total seek time: %f"%self.accessMon.total())
        print("Total disk arm movement: %d"%self.diskMov.total())
        print("Throughput: %f"%(self.byteMon.total()/self.now()))
        print("Total execution time: %f"%self.now())

def runTests():
    """Testing method invoked from user command line option. Tests all the
    algorithms with all possible standard specifications and displays results
    to the command line. Also provides a graphical representation of test 
    results."""
    sims = []
    rects = []
    sys.argv=['DiskSim', '-a', 'FCFS', '-i','150', '-r','10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'SSF', '-i','150', '-r', '10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a','SCAN', '-i','150', '-r', '10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'FCFS', '-i','150', '-r','7200']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'SSF', '-i','150', '-r', '7200']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a','SCAN', '-i','150', '-r', '7200']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'FCFS', '-i','5', '-r','10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'SSF', '-i','5', '-r', '10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a','SCAN', '-i','5', '-r', '10000']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'FCFS', '-i','5', '-r','7200']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a', 'SSF', '-i','5', '-r', '7200']
    sims.append(DiskSim())
    sys.argv=['DiskSim', '-a','SCAN', '-i','5', '-r', '7200']
    sims.append(DiskSim())
    ind = np.arange(4)
    width = 0.30
    fig = plt.figure()
    ax = fig.add_subplot(111)
    f_res = [[]*NUM_TESTS for x in xrange(NUM_ALGS)]
    s_res = [[]*NUM_TESTS for x in xrange(NUM_ALGS)]
    e_res = [[]*NUM_TESTS for x in xrange(NUM_ALGS)]
    for i in range(4):
        f_res[0].append(sims[(NUM_ALGS*i+0)].seekMon.mean())
        f_res[1].append(sims[(NUM_ALGS*i+0)].rotMon.mean())
        f_res[2].append(sims[(NUM_ALGS*i+0)].byteMon.total()/sims[(NUM_ALGS*i+0)].now())
        s_res[0].append(sims[(NUM_ALGS*i+1)].seekMon.mean())
        s_res[1].append(sims[(NUM_ALGS*i+1)].rotMon.mean())
        s_res[2].append(sims[(NUM_ALGS*i+1)].byteMon.total()/sims[(3*i+1)].now())
        e_res[0].append(sims[(NUM_ALGS*i+2)].seekMon.mean())
        e_res[1].append(sims[(NUM_ALGS*i+2)].rotMon.mean())
        e_res[2].append(sims[(NUM_ALGS*i+2)].byteMon.total()/sims[(NUM_ALGS*i+2)].now())
    rects.append(ax.bar((ind+width), f_res[0], width, color='red'))
    rects.append(ax.bar((ind+width*2), s_res[0], width, color='green'))
    rects.append(ax.bar((ind+width*3), e_res[0], width, color='blue'))
    ax.set_ylabel('Milliseconds')
    ax.set_title('Average Seek Time')
    ax.set_xticks(ind+width*2)
    ax.set_xticklabels(('150/10000','150/7200','5/10000','5/7200'))
    ax.legend((rects[0][0],rects[1][0],rects[2][0]), ('FCFS', 'SSF','Elevator/SCAN'))
    plt.show()
    # Rotational Latency Display
    fig = plt.figure()
    ax = fig.add_subplot(111)
    rects.append(ax.bar((ind+width), f_res[1], width, color='red'))
    rects.append(ax.bar((ind+width*2), s_res[1], width, color='green'))
    rects.append(ax.bar((ind+width*3), e_res[1], width, color='blue'))
    ax.set_ylabel('Milliseconds')
    ax.set_title('Average Rotational Latency')
    ax.set_xticks(ind+width*2)
    ax.set_xticklabels(('150/10000','150/7200','5/10000','5/7200'))
    ax.legend((rects[0][0],rects[1][0],rects[2][0]), ('FCFS', 'SSF','Elevator/SCAN'))
    plt.show()
    # Throughput Display
    fig = plt.figure()
    ax = fig.add_subplot(111)
    rects.append(ax.bar((ind+width), f_res[2], width, color='red'))
    rects.append(ax.bar((ind+width*2), s_res[2], width, color='green'))
    rects.append(ax.bar((ind+width*3), e_res[2], width, color='blue'))
    ax.set_ylabel('Milliseconds')
    ax.set_title('Average Throughput')
    ax.set_xticks(ind+width*2)
    ax.set_xticklabels(('150/10000','150/7200','5/10000','5/7200'))
    ax.legend((rects[0][0],rects[1][0],rects[2][0]), ('FCFS', 'SSF','Elevator/SCAN'))
    plt.show()
    sys.exit()

if __name__ == "__main__":
    DiskSim()
```
</details>

### 定义全局变量

```python
# 导入库
from SimPy.Simulation import *
from random import expovariate
import argparse
# 编写的脚本
import ReadQueue
import numpy as np
import matplotlib.pyplot as plt

# 
MOV_TIME = .4
# 
NUM_ALGS = 3
# 
NUM_TESTS = 4
# 每秒读取的字节数
BYTES_PER_SEC = 256
# 扇区大小
SECTORS = 128
# 磁盘请求数
DISK_REQUESTS = 1000
# 平均读取长度
MEAN_READ_LENGTH = 4
# 模拟时长
MAXTIME = 1000000.0
```

### Controller 类

磁盘控制器产生读磁头请求的 DISK_REQUESTS，以用户指定的平均到达时间均匀分布。

```python
class Controller(Process):
    def generate(self, mean):
        for i in range(DISK_REQUESTS):
            req = ReadRequest(name="Request%03d" % (i), sim=self.sim)
            # 产生请求
            self.sim.activate(req, req.request())
            nextReq = random.uniform(0, mean*2)
            if (i == 100):
                self.sim.byteMon.reset()
                self.sim.seekMon.reset()
                self.sim.accessMon.reset()
                self.sim.rotMon.reset()
                self.sim.diskMov.reset()
            yield hold, self, nextReq
```

### ReadRequest 类

ReadRequest 类模拟发送到 arm 的磁盘头读取请求

```python
class ReadRequest(Process):
    def request(self):
        self.trackNum = int(random.uniform(0, 100))
        # 返回：随机 index 分布浮点数
        readLength = expovariate(1.0/MEAN_READ_LENGTH)
        # 读取的总字节数
        self.sim.byteMon.observe(readLength*BYTES_PER_SEC)
        yield request,self,self.sim.head,self.trackNum
        # 磁道的移动距离
        trackDis = abs(self.sim.head.pos - self.trackNum)
        self.sim.diskMov.observe(trackDis)
        # 寻道时间
        seekTime = trackDis * MOV_TIME
        self.sim.seekMon.observe(seekTime)
        # 磁头移动到磁道位置
        self.sim.head.pos = self.trackNum
        # 磁盘旋转时间
        rotTime = (float(SECTORS)/float(self.sim.head.diskRpm)) * readLength
        self.sim.rotMon.observe(rotTime)
        # 请求响应时间 = 寻道时间 + 磁盘旋转时间
        accessTime = seekTime + rotTime
        self.sim.accessMon.observe(accessTime)
        yield hold,self,accessTime
        # 请求完毕，release
        yield release,self,self.sim.head
```

### DiskSim 类

主模拟类。接受用户的命令行输入，指定运行哪种算法、平均到达时间和磁盘最大 rpm 速度。

可选地，可以调用测试模式来测试所有参数，并以图形方式显示它们的结果。

Usage from command line：

```bash
python DiskSim.py -a FCFS -i 25 -r 7000
python DiskSim.py -test True
```

```python
class DiskSim(Simulation):
    def __init__(self):
        # Constructor for DiskSim simulation
        # 1. 创建一个 ArgumentParser 对象。description: 帮助信息
        parser = argparse.ArgumentParser(description='Hard Disk Simulation')
        # 2. 添加参数
        parser.add_argument('-test',default=False,help="""test mode will run through
        the 12 standard test for the disk simulator.""")
        parser.add_argument('-a', choices=['FCFS','SSF','SCAN'],default='SCAN', 
        help="""the algorithm used by the read head when selecting the next 
        sector. Choices are FCFS, SSF, or SCAN. Default is FCFS""")
        parser.add_argument('-i', default=5, type=int, help="""average inter-arrival
        times of disk requests. Will be uniformly distributed; default is 150""")
        parser.add_argument('-r', default=7200, type=int, help="""maximum rotations 
        per minute achieveable by the hard disk. Default is 7200""")
        # 3. 解析参数
        args = parser.parse_args()
        if (args.test):
            runTests()
        # 以上是设置输入参数
        
        # 打印参数
        print("\nAlgorithm: %s"%args.a)
        print("Inter-arrival: %d"%args.i)
        print("Disk RPM: %d"%args.r)
        # SimPy 2 的环境初始化
        self.initialize()
        # SimPy 2 Monitor 监视器
        self.accessMon = Monitor('Access time (Seek + Rotation)', sim=self)
        self.seekMon = Monitor('Seek time', sim=self)
        self.rotMon = Monitor('Rotational latency monitor', sim=self)
        self.diskMov = Monitor('Disk arm movement', sim=self)
        self.byteMon = Monitor('Total bytes read', sim=self)
        # qType 设置队列的类型，作者进行了 readqueue 的重新实现
        # qType 类的实现可以参照 SimPy 中的源码
        self.head = Resource(name="ReadHead",qType= ReadQueue.ReadQ, sim=self)
        # 设置 算法、rpm、磁头位置等参数
        self.head.algorithm = args.a
        self.head.diskRpm = args.r
        self.head.pos = 0
        # Controller 类上面自己定义的产生 disk requests
        controller = Controller('Controller',sim=self)
        # 生成 disk requests
        self.activate(controller, controller.generate(mean=args.i), at=0.0)
        self.simulate(until=MAXTIME)
        print("Average seek time: %f" %self.seekMon.mean())
        print("Average rotational latency: %F"%self.rotMon.mean())
        print("Average access time: %f"%self.accessMon.mean())
        print("Total seek time: %f"%self.accessMon.total())
        print("Total disk arm movement: %d"%self.diskMov.total())
        print("Throughput: %f"%(self.byteMon.total()/self.now()))
        print("Total execution time: %f"%self.now())
```

### SimPy 2 Monitor 类

监控变量的类，即允许收集简单统计信息的变量。
Monitor 是列表的子类，可以对其执行列表操作。使用 `m=Monitor(name='...')` 建立对象。
可以为其指定一个唯一的名称，用于调试和跟踪，以及用于标记图形的 ylab 和 tlab 字符串。

成员函数：

* `setHistogram(self, name, low, high, nbins)`：设置直方图参数；需要在 `getHistogram()` 前调用

* `observe(self, y, t)`：记录 y 和 t

* `reset(self, t)`：重置变量的总和和计数

* `total(self)`：y 的总和

* `mean(self)`：变量的简单平均

* `getHistogram(self)`：返回一个直方图对象；需要在 `setHistogram()` 后调用

* `printHistogram(self, fmt)`：输出直方图，fmt 为格式；需要在 `setHistogram()` 后调用


### runTests() 函数

略


## ReadQueue.py

实现 ReadQueue 队列，根据算法调整

<details>
<summary>完整代码</summary>

```python
from SimPy.Lib import *

MAX_PRIORITY = 200

# Creates enum functionality
def enum(**enums):
    return type('Enum', (), enums)

Dir = enum(FORWARD=1, BACK=2)

class ReadQ(Queue):
    """ReadQueue simulates the internal request queue'ing capabilities of the
    heard disk read head. Queues used and priority depend on specified 
    algorithm"""
    def __init__(self, res, moni):
        """Constructor for ReadQ data structure"""
        self.direction = Dir.FORWARD
        Queue.__init__(self, res, moni)

    def enter(self, obj):
        """Add a read request to the waiting queue. Called by Simulator."""
        self.append(obj)
        if self.monit:
            self.moni.observe(len(self),t = self.moni.sim.now())
    
    def scanDistance(self, read):
        """Prioritizes reads according to the SCAN/ELEVATOR algorithm"""
        if (self.direction == Dir.FORWARD):
            if (self.resource.pos > read.trackNum):
                return (MAX_PRIORITY-read.trackNum)
            return read.trackNum
        elif (self.direction == Dir.BACK):
            if (self.resource.pos < read.trackNum):
                return (MAX_PRIORITY+read.trackNum)
            return -read.trackNum
    
    def leave(self):
        """Determines which read to service next according to algorithm, and 
        returns it. Changes the direction of reads when appropriate for SCAN
        algorithm. Called by Simulator."""
        if (self.resource.algorithm == 'SSF'):
            self.sort(key=lambda read: abs(read.trackNum-self.resource.pos))
        elif (self.resource.algorithm == 'SCAN'):
            if (self.direction == Dir.FORWARD):
                self.sort(key=self.scanDistance)
                if (self.resource.pos > self[0].trackNum):
                    self.direction = Dir.BACK
            elif (self.direction == Dir.BACK):
                self.sort(key=self.scanDistance)
                if (self.resource.pos < self[0].trackNum):
                    self.direction = Dir.FORWARD
        ele = self.pop(0)
        if self.monit:
            self.moni.observe(len(self),t = self.moni.sim.now())
        return ele
```
</details>

### 定义全局变量

```python
# 最大优先级 200
MAX_PRIORITY = 200
```

### 定义枚举

定义了一个函数。可以用 Enum 和类来实现枚举数据类型：`class MyEnum(Enum):`。

```python
# 磁头方向
def enum(**enums):
    return type('Enum', (), enums)

Dir = enum(FORWARD=1, BACK=2)
```

### ReadQ 类

ReadQueue 模拟磁盘读磁头的内部请求排队功能。使用的队列和优先级取决于指定的算法

```python
# 仿照 SimPy 的 FIFO 类
class ReadQ(Queue):
    """
    ReadQueue 模拟磁盘读磁头的内部请求排队功能。使用的队列和优先级取决于指定的算法
    """
    def __init__(self, res, moni):
        # 初始磁头方向为前进
        self.direction = Dir.FORWARD
        Queue.__init__(self, res, moni)

    def enter(self, obj):
        """
        加入一个 read request 到 wait queue 中
        由 Simulator 调用
        """
        self.append(obj)
        if self.monit:
            self.moni.observe(len(self),t = self.moni.sim.now())
    
    # 自定义的函数
    def scanDistance(self, read):
        """
        根据 SCAN/ELEVATOR 算法确定读取优先级
        数值越小、优先级越高
        如果要读取的 trackNum 与当前磁头移动方向一致，优先级更高；否则优先级更低
        """
        # trackNum ~ (0, 100)
        if (self.direction == Dir.FORWARD):
            if (self.resource.pos > read.trackNum):
                # MAX_PRIORITY-trackNum ~ (100, 200)
                return (MAX_PRIORITY-read.trackNum)
            return read.trackNum
        elif (self.direction == Dir.BACK):
            if (self.resource.pos < read.trackNum):
                # MAX_PRIORITY+trackNum ~ (200, 300)
                return (MAX_PRIORITY+read.trackNum)
            return -read.trackNum
    
    def leave(self):
        """
        根据算法确定下一个要服务的 read 并返回
        在 SCAN 算法的情况下更改读取方向
        由 Simulator 调用
        """
        # FCFS 直接根据进入 Queue 顺序
        # SSF 根据 trackNum 距离磁头的位置进行 read 排序
        if (self.resource.algorithm == 'SSF'):
            self.sort(key=lambda read: abs(read.trackNum-self.resource.pos))
        # SCAN 根据磁头方向以及 scanDistance() 中确定的优先级进行 read 排序
        elif (self.resource.algorithm == 'SCAN'):
            if (self.direction == Dir.FORWARD):
                self.sort(key=self.scanDistance)
                # self[0] ==> self.__getItem__(0)
                # ReadQueue 类继承自 Queue 类继承自 list 类，list 类中有 __getItem__() 函数
                # 这里不是达到磁盘的 track 一端再反向，
                # 而是当磁头位置大于/小于数值最小（优先级最高）时反向
                if (self.resource.pos > self[0].trackNum):
                    # 反向
                    self.direction = Dir.BACK
            elif (self.direction == Dir.BACK):
                self.sort(key=self.scanDistance)
                if (self.resource.pos < self[0].trackNum):
                    # 反向
                    self.direction = Dir.FORWARD
        ele = self.pop(0)
        if self.monit:
            self.moni.observe(len(self),t = self.moni.sim.now())
        return ele
```