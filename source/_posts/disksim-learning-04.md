---
title: DiskSim 学习（四）：参数文件
tags:
  - DiskSim
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - DiskSim
comments: false
cover: false
date: 2023-11-08 08:17:11
---

DiskSim 的 `.parv` 参数文件

<!-- more -->

DiskSim 使用 libparam 输入参数文件。在参数文件中有三种内容：

* `blocks delimited by {}`，块的定义
* `instantiations`，实例化
* `topology specifications`，拓扑结构说明

一个 block 由多个 "name=value" 赋值组成。名称可能包含空格，并且区分大小写。
值可以是整数（包括前缀为 0x 的十六进制）、浮点数、字符串、块和由 [] 分隔的列表。

组件不能在定义前引用。每个参数文件必须包含 `Global` 和 `States` 块，
使用合成跟踪发生器仿真还必须定义 `proc` 和 `synthio` 块
（disksim_pf { }：Process-Flow Parameters 合成负载的一些参数，disksim_synthio { }：SyntheticWorkloads 合成负载）。

在一个典型的配置中，接着还会再定义一定的总线（bus）、控制器（controller）、和输入输出驱动（iodriver），
然后定义或引用一些存储设备的说明，之后实例化，最后模拟存储仿真的拓扑结构规范定义这些组件之间的互连。

## atlas_III.parv 参数文件

主要包括以下几个部分：

* Global block
* Stats block 
* 具体设备参数设定
  * device drivers
  * buses
  * controllers
  * storage devices
* 实例化
* 系统拓扑结构
* 设置 workload

### Global block

```python
disksim_global Global { 
# 每次模拟开始时会由 randomnumber generator 产生一个 initial seed，决定系统 configuration。
# 若希望实验可还原，可使用相同的 Init Seed
Init Seed = 42,
# 决定系统的 workload 的初始随机数，
# 当保持 Init Seed 不变而改变 Real Seed 时，可得到相同系统 configuration 下不同 workload 的实验结果 
Real Seed = 42,
# Statistic warm-up period = 0.0 seconds,
Stat definition file = statdefs
}
```

### Stats block

包含五个子块：iodriver、bus、ctlr、device、process flow，参数全由 0 或 1 表示。

1 和 0 分别表示这些参数是否需要 print 到输出文件中。

```python
disksim_stats Stats {
 iodriver stats = disksim_iodriver_stats {
  Print driver size stats = 1,
  Print driver locality stats = 0,
  Print driver blocking stats = 0,
  Print driver interference stats = 0,
  Print driver queue stats = 1,
  Print driver crit stats = 1,
  Print driver idle stats = 1,
  Print driver intarr stats = 1,
  Print driver streak stats = 1,
  Print driver stamp stats = 1,
  Print driver per-device stats = 1 },
 bus stats = disksim_bus_stats {
  Print bus idle stats = 1,
  Print bus arbwait stats = 1 },
 ctlr stats = disksim_ctlr_stats {
  Print controller cache stats = 1,
  Print controller size stats = 1,
  Print controller locality stats = 1,
  Print controller blocking stats = 1,
  Print controller interference stats = 1,
  Print controller queue stats = 1,
  Print controller crit stats = 1,
  Print controller idle stats = 1,
  Print controller intarr stats = 1,
  Print controller streak stats = 1,
  Print controller stamp stats = 1,
  Print controller per-device stats = 1 }, 
 device stats = disksim_device_stats {
  Print device queue stats = 0,
  Print device crit stats = 0,
  Print device idle stats = 0,
  Print device intarr stats = 0,
  Print device size stats = 0,
  Print device seek stats = 1,
  Print device latency stats = 1,
  Print device xfer stats = 1,
  Print device acctime stats = 1,
  Print device interfere stats = 0,
  Print device buffer stats = 1 },
 process flow stats = disksim_pf_stats {
  Print per-process stats =  1,
  Print per-CPU stats =  1,
  Print all interrupt stats =  1,
  Print sleep stats =  1
 }
} # end of stats block
```

### 具体设备参数设定

```python
# Device Driver
disksim_iodriver DRIVER0 {
 type = 1,
 # 决定每个 request 的访问时间和队列时间是否一致，还是由下面 disk 根据自身情况自行计算
 Constant access time = 0.0,
 
 Scheduler = disksim_ioqueue {
  # Scheduling 算法，如何选择下一个将要处理的 request，1 代表“先来先服务”
  Scheduling policy = 1,
  Cylinder mapping strategy = 1,
  Write initiation delay = 0.0,
  Read initiation delay = 0.0,
  Sequential stream scheme = 0,
  Maximum concat size = 128,
  Overlapping request scheme = 0,
  Sequential stream diff maximum = 0,
  Scheduling timeout scheme = 0,
  Timeout time/weight = 6,
  Timeout scheduling = 4,
  Scheduling priority scheme = 0,
  Priority scheduling = 4
 }, # end of Scheduler
 # 设备驱动是否在任何时候都允许两个或者更多的请求存在存储子系统中
 Use queueing in subsystem = 1
} # end of DRV0 spec
```

```python
# Buses
disksim_bus BUS0 {
 # bus 类型：1，独占式总线；2，共享式总线
 type = 1,
 Arbitration type = 1,
 Arbitration time = 0.0,
 Read block transfer time = 0.0,
 Write block transfer time = 0.0,
 Print stats =  0
} # end of BUS0 spec

disksim_bus BUS1 {
 type = 1,
 Arbitration type = 1,
 Arbitration time = 0.0,
 Read block transfer time = 0.0,
 Write block transfer time = 0.0,
 Print stats =  1
} # end of BUS1 spec
```

```python
# controller
disksim_ctlr CTLR0 {
 type = 1,
 Scale for delays = 0.0,
 Bulk sector transfer time = 0.0,
 Maximum queue length = 0,
 Print stats =  1
} # end of CTLR0 spec
```

```python
# QUANTUM_QM39100TD-SW
# source 类似于预处理指令
source atlas_III.diskspecs
```


### 实例化

实例化的语法格式：`instantiate <name list> as <instance name>`

```python
# component instantiation
instantiate [ statfoo ] as Stats
instantiate [ bus0 ] as  BUS0
instantiate [ bus1 ] as  BUS1
instantiate [ disk0 ] as  QUANTUM_QM39100TD-SW
instantiate [ driver0 ] as  DRIVER0
instantiate [ ctlr0 ] as  CTLR0
```

### 拓扑结构

```python
# system topology
topology disksim_iodriver driver0 [
   disksim_bus bus0 [ 
      disksim_ctlr ctlr0 [ 
         disksim_bus bus1 [ 
            disksim_disk disk0 []
         ] # end of bus1
      ] # end of ctlr0
   ] # end of bus0
] # end of system topology 

# no syncsets

disksim_logorg org0 {
   Addressing mode = Parts,
   Distribution scheme = Asis,
   Redundancy scheme = Noredun,
   devices = [ disk0 ],
   Stripe unit  =  17783250,
   Synch writes for safety =  0,
   Number of copies =  2,
   Copy choice on read =  6,
   RMW vs. reconstruct =  0.5,
   Parity stripe unit =  64,
   Parity rotation type =  1,
   Time stamp interval =  0.000000,
   Time stamp start time =  60000.000000,
   Time stamp stop time =  10000000000.000000,
   Time stamp file name =  stamps
} # end of logorg org0 spec
```


### 设置 Workload

下面设置 workload，可有多个 generator，每个 generator 每过一段 think time 后发出一个 request，
包括 time-criticalreques（generator 需等上一个 request 完成再进入下一个 think time）

```python
disksim_pf Proc {
   Number of processors =  1,
   Process-Flow Time Scale =  1.0
} # end of process flow spec

disksim_synthio Synthio {
   Number of I/O requests to generate =  10000,
   Maximum time of trace generated  = 1000.0,
   System call/return with each request = 0,
   Think time from call to request =  0.0,
   Think time from request to return =  0.0,
   Generators = [
   disksim_synthgen { # generator 0 
      Storage capacity per device  = 17783250,
      devices = [ disk0 ], 
      Blocking factor =  8,
      Probability of sequential access =  0.0,
      Probability of local access =  0.0,
      Probability of read access =  0.66,
      Probability of time-critical request = 1.0,
      Probability of time-limited request = 0.0,
      Time-limited think times  = [ normal, 30.0, 100.0  ],
      General inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Sequential inter-arrival times  = [ normal, 0.0, 0.0  ],
      Local inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Local distances  = [ normal, 0.0, 40000.0  ],
      Sizes  = [ exponential, 0.0, 8.0  ]
   }, # end of generator 0 
   disksim_synthgen { # generator 1
      Storage capacity per device  = 17783250,
      devices = [ disk0 ], 
      Blocking factor =  8,
      Probability of sequential access =  0.0,
      Probability of local access =  0.0,
      Probability of read access =  0.66,
      Probability of time-critical request = 1.0,
      Probability of time-limited request = 0.0,
      Time-limited think times  = [ normal, 30.0, 100.0  ],
      General inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Sequential inter-arrival times  = [ normal, 0.0, 0.0  ],
      Local inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Local distances  = [ normal, 0.0, 40000.0  ],
      Sizes  = [ exponential, 0.0, 8.0  ]
   }, # end of generator 1 
   disksim_synthgen { # generator 2 
      Storage capacity per device  = 17783250,
      devices = [ disk0 ], 
      Blocking factor =  8,
      Probability of sequential access =  0.0,
      Probability of local access =  0.0,
      Probability of read access =  0.66,
      Probability of time-critical request = 1.0,
      Probability of time-limited request = 0.0,
      Time-limited think times  = [ normal, 30.0, 100.0  ],
      General inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Sequential inter-arrival times  = [ normal, 0.0, 0.0  ],
      Local inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Local distances  = [ normal, 0.0, 40000.0  ],
      Sizes  = [ exponential, 0.0, 8.0  ]
   }, # end of generator 2 
   disksim_synthgen { # generator 3 
      Storage capacity per device  = 17783250,
      devices = [ disk0 ], 
      Blocking factor =  8,
      Probability of sequential access =  0.0,
      Probability of local access =  0.0,
      Probability of read access =  0.66,
      Probability of time-critical request = 1.0,
      Probability of time-limited request = 0.0,
      Time-limited think times  = [ normal, 30.0, 100.0  ],
      General inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Sequential inter-arrival times  = [ normal, 0.0, 0.0  ],
      Local inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Local distances  = [ normal, 0.0, 40000.0  ],
      Sizes  = [ exponential, 0.0, 8.0  ]
   }, # end of generator 3 
   disksim_synthgen { # generator 4 
      Storage capacity per device  = 17783250,
      devices = [ disk0 ], 
      Blocking factor =  8,
      Probability of sequential access =  0.0,
      Probability of local access =  0.0,
      Probability of read access =  0.66,
      Probability of time-critical request = 1.0,
      Probability of time-limited request = 0.0,
      Time-limited think times  = [ normal, 30.0, 100.0  ],
      General inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Sequential inter-arrival times  = [ normal, 0.0, 0.0  ],
      Local inter-arrival times  = [ exponential, 0.0, 0.0  ],
      Local distances  = [ normal, 0.0, 40000.0  ],
      Sizes  = [ exponential, 0.0, 8.0  ]
   } # end of generator 4 
   ] # end of generator list 
} # end of synthetic workload spec
```