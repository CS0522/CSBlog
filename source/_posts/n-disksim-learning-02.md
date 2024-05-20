---
title: 【学习笔记】DiskSim 学习（二）：简单使用
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
date: 2023-11-06 08:17:11
---

DiskSim 的简单使用方法以及命令的各个参数

<!-- more -->

DiskSim 包含许多存储元件的模型：`device drivers, buses, controllers, adapters, disk drives`。

在 `disksim-4.0/valid` 和 `disksim-4.0/ssdmodel/valid/` 下都有 `runvalid` 脚本，里面有可以参考的运行命令示例。

整个系统里面有两个文件包含可以运行的主函数： `disksim.c` 和 `syssim_driver.c`。

## disksim.c

`disksim-4.0/src` 目录下的 `disksim` 为可执行程序。运行的命令格式为：

```bash
disksim <parfile> <outfile> <tracetype> <tracefile> <synthgen> [par_override…]
# 示例
disksim atlas_III.parv atlas_III.outv validate atlas_III.trace 0
```

### parfile

参数文件，包括仿真器各个参数的配置。后续文章继续学习。

### outfile

输出文件，outfile 的项目内容可在 parfile 中设置，以去掉不需要的内容。

### tracetype

确定输入 trace 的格式。比如：ASCII HPL HPL2 DEC VALIDATE RAW ATABUS IPEAK P OSTGRES EMCSYMM DEFAULT(ASCII) 等等。

Disksim 本身支持的 trace 类型是有限的，如果不支持实验使用的 trace，则需要修改 disksim 源程序以添加新的 trace 类型，添加方法如下： 

* 在 disksim_global.h 中预定义 trace 类型常量
* 在 disksim_iotrace.c 文件的 iotrace_set_format 函数中添加相应的 if 判断语句
* 在 disksim_iotrace.c 文件中仿照 iotrace_ascii_get_ioreq_event 函数添加 iotrace_spc_get_ioreq_event 函数处理相应的 trace 文件。如果 trace 文件中包含一定的头信息，则需要在 disksim_iotrace.c 文件中添加 iotrace_xxxx_initialize_file 函数，并在 iotrace_initialize_file 函数中调用。

具体方法参照官方手册。后续文章继续学习。

### tracefile

输入的 trace 文件。如果为 0 的话表示使用系统自动生成的 trace 数据。如果这个参数为 0，那么 synthgen 参数必须要大于 0，表示开启合成负载。

trace 每一行包含 5 个参数值（用空格隔开）来描述一个磁盘请求:

* 请求到达时间（double 型，表示从模拟开始到请求发生的时间，必须按时间上升的顺序
* 设备号（int 型，指定设备号，如果有设备映射，被加到该值）
* 块号（int 型，表示请求的第一个设备地址，其值为逻辑设备可访问的单元）
* 请求大小（int 型）
* 请求标志（十六进制整数，有效值在 disksim_global.h 中定义）

### synthgen

决定合成负载部分的模拟器是否打开（除 0 外的数表示开启）

当 synthgen 参数不为 0 时，那么 tracefile 参数就需要设置为 0，synthgen 的值就表示了用多少个 generator 来生成 trace，一个 generator 代表一个产生io请求的进程。每个 generator 的行为要在参数文件里面设置

模块可以配置为生成多钟合成的工作负载，与磁盘位置访问和请求到达时间相关。每一个合成器在简单系统模型里作为一个进程，在一定的“think time”后发出I/O请求，并等待完成。

对于特定的需求，实验时需要调整参数文件中 disksim_synthgen 配置或修改 disksim_synthetic.c 文件中的函数 synthio_generatenextio 以满足特定的请求特征（如访问的hot/cold 特征，zipfan 分布等），然后编译运行，获得实验结果。

Disksim提供了 uniform，normal，possion，expontenial，twovalue 等概率分布，可以灵活运用这些分布生成具有特定特征的请求，如符合 zipfan 分布的请求

### par_override

允许默认参数值或者 parfile 文件中的参数值替换为命令行指定的值。

parameter overrides 的格式为 `(component, param name, param value)` 三元组：

```bash
<component> <parameter> <new value>
```

* `component` 是需要被覆盖参数的目标组件。支持多个修改和通配符。用 `""` 包裹。如：`"disk0 .. disk5"`、`"driver*"`（`*` 通配 0 或多个`数字`而不是字符）

* `parameter` 是需要被覆盖的参数。最好用 `""` 包裹，因为可能会有空格。如：`"Scheduler:Scheduling policy"`

* `new value` 是新的值


### Examples

```bash
# 格式
disksim <parfile> <outfile> <tracetype> <tracefile> <synthgen> [par_override…]

disksim parms.1B stdout ascii t.Jan6 0 "disk1 .. disk16" "Segment size (in blks)" 64 "disk*" "Scheduler:Scheduling policy" 4
```

运行后：

* 从文件 `parms.1B` 中读取初始化参数

* output 发送至 `stdout`

* 读取 `ascii` 格式的 trace 文件 `t.Jan6`

* 无合成工作负载

* `disk1 ~ disk16` 的 `cache segment size` 参数值被覆盖为 `64`

* 所有匹配 `disk*` 的组件的 `scheduling plicy` 值被覆盖为 `4`（对应为 SSTF 调度算法）