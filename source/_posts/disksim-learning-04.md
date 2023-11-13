---
title: 【学习笔记】DiskSim 学习（四）：参数文件
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

DiskSim 的 .parv 参数文件

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

磁盘阵列的数据组织是在 `logorg` 块中描述的。每一个请求都必须属于一个 `logorg`，所以至少有一个 `logorg` 块被定义。

设备的 `Rotational syncrhonization` 可以选择性的在 `syncset` 块中定义。

`adjusting the time scale` 和 `remapping requests from a trace` 可以在 `iosim` 块中定义。

## synthraid5.parv 参数文件

主要包括以下几个部分：

* Global block
* Stats block 
* 具体设备参数设定
  * device drivers
  * buses
  * controllers
  * storage devices
* 实例化
* 拓扑结构
* RAID 磁盘阵列
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
 Print driver locality stats = 1,
 Print driver blocking stats = 1,
 Print driver interference stats = 1,
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
 Print device queue stats = 1, 
 Print device crit stats = 1,
 Print device idle stats = 1,
 Print device intarr stats = 1,
 Print device size stats = 1,
 Print device seek stats = 1,
 Print device latency stats = 1,
 Print device xfer stats = 1,
 Print device acctime stats = 1,
 Print device interfere stats = 1,
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

> 注：以下设备参数取自 `atlas_III.parv` 参数文件

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
# HP_C3323A
# source 类似于预处理指令
source hp_c3323a.diskspecs
```


### 实例化

实例化的语法格式：`instantiate <name list> as <instance name>`

```python
# component instantiation
instantiate [ statfoo ] as Stats
instantiate [ ctlr0 .. ctlr8 ] as CTLR0
instantiate [ bus0 ] as BUS0
instantiate [ disk0 .. disk8 ] as HP_C3323A
instantiate [ driver0 ] as DRIVER0
instantiate [ bus1 .. bus9 ] as BUS1
# end of component instantiation
```

### 拓扑结构

```python
# system topology
topology disksim_iodriver driver0 [
     disksim_bus bus0 [ 
          disksim_ctlr ctlr0 [ 
               disksim_bus bus1 [ 
                    disksim_disk disk0 []
                    # end of bus1
               ]
               # end of ctlr0
          ],
          disksim_ctlr ctlr1 [ 
               disksim_bus bus2 [ 
                    disksim_disk disk1 []
                    # end of bus2
               ]
               # end of ctlr1
          ],
          disksim_ctlr ctlr2 [ 
               disksim_bus bus3 [ 
                    disksim_disk disk2 []
                    # end of bus3
               ]
               # end of ctlr2
          ],
          disksim_ctlr ctlr3 [ 
               disksim_bus bus4 [ 
                    disksim_disk disk3 []
                    # end of bus4
               ]
               # end of ctlr3
          ],
          disksim_ctlr ctlr4 [ 
               disksim_bus bus5 [ 
                    disksim_disk disk4 []
                    # end of bus5
               ]
               # end of ctlr4
          ],
          disksim_ctlr ctlr5 [ 
               disksim_bus bus6 [ 
                    disksim_disk disk5 []
                    # end of bus6
               ]
               # end of ctlr5
          ],
          disksim_ctlr ctlr6 [ 
               disksim_bus bus7 [ 
                    disksim_disk disk6 []
                    # end of bus7
               ]
               # end of ctlr6
          ],
          disksim_ctlr ctlr7 [ 
               disksim_bus bus8 [ 
                    disksim_disk disk7 []
                    # end of bus8
               ]
               # end of ctlr7
          ],
          disksim_ctlr ctlr8 [ 
               disksim_bus bus9 [ 
                    disksim_disk disk8 []
                    # end of bus9
               ]
               # end of ctlr8
          ]
          # end of bus0
     ]
     # end of system topology
]
```

### RAID 磁盘阵列

```python
disksim_syncset sync0 { 
   devices = [ disk0 .. disk8 ] 
}

# 重要！！！这里应该涉及到磁盘阵列的数据分布

# Disk Array Data Organizations（磁盘阵列逻辑数据组织形式）
# 每一种逻辑结构都可以在 logory 块中配置
# RAID-5
disksim_logorg org0 {
    # 逻辑数据组织的编址方式（值为：Array（编成一个统一逻辑设备）或 Parts）
    Addressing mode = Array,
    # 数据分布策略，体现负载均衡能力（值为：Striped，Random，N.B. 或 Ideal）
    Distribution scheme = Striped,
    # 冗余方案（值为：Noredun，Shadowed，Parity_disk 或 Parity_rotated）
    Redundancy scheme = Parity_rotated,
    # 当前逻辑组织中包含的设备名称列表
    devices = [ disk0 .. disk8 ],
    # stripe 单元的大小
    Stripe unit  =  64,
    Synch writes for safety =  0,
    # 每个数据磁盘的备份数（仅当 Redundancy scheme = shadowed 时才有效）
    Number of copies =  2,
    # 哪个副本负责响应（仅当 Redundancy scheme = shadowed 时才有效）
    Copy choice on read =  6,
    RMW vs. reconstruct =  0.5,
    # stripe 单元的大小（仅当 Redundancy scheme = Parity_rotated 时才有效）
    Parity stripe unit =  64,
     # parity 在磁盘中旋转的方式（仅当 Redundancy scheme = Parity_rotated 时才有效）
    Parity rotation type =  1,
    # time stamps 之间的间隔
    Time stamp interval =  0.000000,
     # 第一个 time stamp 的模拟时间（相对于模拟开始的时间）
    Time stamp start time =  60000.000000,
     # 最后一个 time stamp 的模拟时间（相对于模拟开始的时间）
    Time stamp stop time =  10000000000.000000,
    Time stamp file name =  stamps
} # end of logorg org0 spec
```


### 设置 Workload

下面设置 workload，可有多个 generator，每个 generator 每过一段 think time 后发出一个 request，
包括 time-criticalreques（generator 需等上一个 request 完成再进入下一个 think time）

```python
disksim_pf Proc {
   Number of processors =  5,
   Process-Flow Time Scale =  1.0
} # end of process flow spec

disksim_synthio Synthio {
   Number of I/O requests to generate =  10000,
   Maximum time of trace generated  =  1000.0,
   System call/return with each request =  0,
   Think time from call to request =  0.0,
   Think time from request to return =  0.0,
   Generators = [
     disksim_synthgen { # generator 0 
       Storage capacity per device  =  16448064,
       devices = [ org0 ], 
       Blocking factor =  8,
       Probability of sequential access =  0.2,
       Probability of local access =  0.3,
       Probability of read access =  0.66,
       Probability of time-critical request =  0.1,
       Probability of time-limited request =  0.3,
       Time-limited think times  = [ normal, 30.0, 100.0  ],
       General inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Sequential inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local distances  = [ normal, 0.0, 40000.0  ],
       Sizes  = [ exponential, 0.0, 8.0  ]
     }, # end of generator 0 
     disksim_synthgen { # generator 0 
       Storage capacity per device  =  16448064,
       devices = [ org0 ], 
       Blocking factor =  8,
       Probability of sequential access =  0.2,
       Probability of local access =  0.3,
       Probability of read access =  0.66,
       Probability of time-critical request =  0.1,
       Probability of time-limited request =  0.3,
       Time-limited think times  = [ normal, 30.0, 100.0  ],
       General inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Sequential inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local distances  = [ normal, 0.0, 40000.0  ],
       Sizes  = [ exponential, 0.0, 8.0  ]
     }, # end of generator 0 
     disksim_synthgen { # generator 0 
       Storage capacity per device  =  16448064,
       devices = [ org0 ], 
       Blocking factor =  8,
       Probability of sequential access =  0.2,
       Probability of local access =  0.3,
       Probability of read access =  0.66,
       Probability of time-critical request =  0.1,
       Probability of time-limited request =  0.3,
       Time-limited think times  = [ normal, 30.0, 100.0  ],
       General inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Sequential inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local distances  = [ normal, 0.0, 40000.0  ],
       Sizes  = [ exponential, 0.0, 8.0  ]
     }, # end of generator 0 
     disksim_synthgen { # generator 0 
       Storage capacity per device  =  16448064,
       devices = [ org0 ], 
       Blocking factor =  8,
       Probability of sequential access =  0.2,
       Probability of local access =  0.3,
       Probability of read access =  0.66,
       Probability of time-critical request =  0.1,
       Probability of time-limited request =  0.3,
       Time-limited think times  = [ normal, 30.0, 100.0  ],
       General inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Sequential inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local distances  = [ normal, 0.0, 40000.0  ],
       Sizes  = [ exponential, 0.0, 8.0  ]
     }, # end of generator 0 
     disksim_synthgen { # generator 0 
       Storage capacity per device  =  16448064,
       devices = [ org0 ], 
       Blocking factor =  8,
       Probability of sequential access =  0.2,
       Probability of local access =  0.3,
       Probability of read access =  0.66,
       Probability of time-critical request =  0.1,
       Probability of time-limited request =  0.3,
       Time-limited think times  = [ normal, 30.0, 100.0  ],
       General inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Sequential inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local inter-arrival times  = [ exponential, 0.0, 10.0  ],
       Local distances  = [ normal, 0.0, 40000.0  ],
       Sizes  = [ exponential, 0.0, 8.0  ]
     } # end of generator 0 
   ] # end of generator list 
} # end of synthetic workload spec
```