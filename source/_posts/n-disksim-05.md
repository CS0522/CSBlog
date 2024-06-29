---
title: 【学习笔记】DiskSim 学习（五）：RAID 磁盘数据分布策略
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
date: 2023-11-10 08:17:11
---

DiskSim 的 RAID 磁盘数据分布策略的参数设置以及源码修改

<!-- more -->

## 原论文中 Disk Array Data Organizations 部分

因为磁盘阵列 Data Organizations 可以用于多个组件（例如，设备驱动程序和智能存储控制器），
逻辑地址映射被实现为单独的模块（`logorg` 模块），并通过适当与其他组件联合。
由 `data distribution scheme` 和 `redundancy mechanism` 构成的 `logorg` 模块支持大多数逻辑数据组织。

`logorg` 模块将一个新到达的 logical request 转换为合适的一组 physical accesses（以 components 的视角）（如使用了 disk striping 和/或基于副本的冗余），
并返回其中的子集。

一个 `logorg` 模块的实例可以由多个 parameters 配置，parameters 介绍参考后两节。


## Manual 中 Disk Array Data Organizations 部分

DiskSim 可以模拟各种 logical organization，包括 striping 和各种 RAID 体系结构。尽管 DiskSim 被组织为允许在<font color="red">系统级别</font>（即，在设备驱动程序的前端）和<font color="red">控制器级别</font>进行这样的组织，但在 version1.0 中仅支持系统级别的组织。每个 logical organization 都配置在一个 `logorg` 模块中。

参数含义：

### Addressing mode

指定了 logical data organization 的寻址方式。

* `Array`：整个 logical organization 只有一个逻辑设备编号
* `Parts`：后端存储设备的寻址方式就像没有逻辑组织一样，并且请求被适当地重新映射

### Distribution scheme

指定了数据分发方案（与冗余方案正交：可以单独使用，或者组合使用）。

* `Asis`：没有重新映射
* `Striped`：数据在磁盘上分条
* `Random`：为每个请求选择一个随机磁盘。
* `N.B.`：只能与用于负载平衡实验的 constant access-time 磁盘一起使用
* `Ideal`：理想化的数据分布（从负载平衡的角度来看）应该通过以循环方式向磁盘分配请求来模拟

最后两种方案没有对实际数据布局进行建模。特别是，对同一块的两个请求通常会发送到不同的设备。这些数据分布方案对于研究各种负载平衡技术非常有用。

### Redundancy scheme

指定了冗余方案（与数据分发方案正交）。

* `Noredun`：不采用冗余
* `Shadowed`：维护每个数据磁盘的一个或多个副本
* `Parity disk`：其中一个物理磁盘包含根据其他磁盘的内容计算的奇偶校验信息
* `Parity rotated`：一个磁盘容量大小（分布在所有磁盘上），专用于保存奇偶校验信息，以保护 N 磁盘组织中其他 N-1 个磁盘的信息

### Stripe unit

指定条带单元的大小。0 表示细粒度条带化（例如，位或字节条带化），其中逻辑组织中的所有数据磁盘包含每个可寻址数据单元的相等部分。

### Parity stripe unit

指定用于奇偶校验旋转冗余方案的条带单元大小。对于其他方案，此参数将被忽略。奇偶校验条带单元大小不必等于条带单元的大小，但其中一个必须是另一个的倍数。

### Parity rotation type

指定如何在逻辑组织的磁盘之间轮换奇偶校验。4 个选项：1 - 左对称、2 - 左不对称、3 - 右不对称、4 - 右对称。除非选择了奇偶校验旋转冗余，否则此参数将被忽略。

## parv 参数文件中 RAID 参数

> synthraid5.parv

```python
disksim_syncset sync0 { 
   devices = [ disk0 .. disk8 ] 
}

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


## DiskSim src 源码部分

四个文件：

* `disksim_logorg_param.h`：加载 logorg 模块参数的头文件
* `disksim_logorg_param.c`：加载 logorg 模块参数的源文件
* `disksim_logorg.h`：定义各种结构体和函数
* `disksim_logorg.c`：logorg 模块的实现


### 设置参数流程

以 Distribution Scheme 参数为例：

1. `disksim_logorg_param.h`: extern void * DISKSIM_LOGORG_loaders[]
2. `disksim_logorg_param.c`: void * DISKSIM_LOGORG_loaders[] = {.. (void *)DISKSIM_LOGORG_DISTRIBUTION_SCHEME_loader, ..}
3. `disksim_logorg_param.c`: static void DISKSIM_LOGORG_DISTRIBUTION_SCHEME_loader(struct logorg * result, char *s)
4. `disksim_logorg.c`: int logorg_distn(logorg *result, char *s)

<details>
<summary>以 Distribution Scheme 参数为例</summary>

```c
// 1. disksim_logorg_param.h: 34
extern void * DISKSIM_LOGORG_loaders[];


// 2. disksim_logorg_param.c: 172
void * DISKSIM_LOGORG_loaders[] = {
...,
(void *)DISKSIM_LOGORG_DISTRIBUTION_SCHEME_loader,
...
}


// 3. disksim_logorg_param.c: 18
static void DISKSIM_LOGORG_DISTRIBUTION_SCHEME_loader(struct logorg * result, char *s) { 
if (! (!logorg_distn(result, s))) { // foo 
 } 

}


// 4. disksim_logorg.c: 1529
int logorg_distn(logorg *result, char *s) {
  if(!strcmp(s, "Asis")) { 
    result->maptype = ASIS;
  } 
  else if(!strcmp(s, "Striped")) { 
    result->maptype = STRIPED;
  } 
  else if(!strcmp(s, "Random")) { 
    fprintf(stderr, "*** warning: Random logorg distribution is only "
	    "to be used with constant access-time disks "
	    "for load-balancing experiments \n");

    result->maptype = RANDOM;
  } 
  else if(!strcmp(s, "Ideal")) { 
    result->maptype = IDEAL;
  } 
  else { 
    fprintf(stderr, "*** error: %s is not a valid argument for logorg addressing mode\n", s); 
    return -1; 
  }
  return 0;
}
```
</details>


### 开始运行到发请求至 RAID 流程

初始化流程：

1. `disksim_main.c`: disksim_setup_disksim(argc, argv);
2. `disksim.c`: initialize();
3. `disksim.c`: io_initialize(val);
4. `disksim_iosim.c`: iodriver_initialize(standalone);
5. `disksim_iodriver.c`: logorg_initialize(...);
6. `disksim_logorg.c: 1104`: logorg_initialize(...) 声明


RAID 请求重新映射流程：

1. `disksim_main.c`: disksim_run_simulation();
2. `disksim.c`: disksim_simulate_event(event_count);
3. `disksim.c`: io_internal_event(ioreq_event *curr);
4. `disksim_iosim.c`: iodriver_request(0, curr);
5. `disksim_iodriver.c`: logorg_maprequest(sysorgs, numsysorgs, curr);
6. `disksim_logorg.c:695`: logorg_maprequest(...) 声明


### disksim_logorg.h

主要关注与 Distribution Scheme 和 Redundancy 有关的代码。

很多宏定义（用整型来表示参数的各个值）和预定义的结构体。主要是 `logorgstat` 和 `logorg` 结构体。
`logorgstat` 包含 RAID 的状态，`logorg` 包含 RAID 各种参数设置。

<details>
<summary>logorgstat 结构体</summary>

```c
typedef struct {
   double	outtime;
   double	runouttime;
   int		outstanding;
   int		readoutstanding;
   int		maxoutstanding;
   double	nonzeroouttime;
   int		reads;
   int		gens[NUMGENS];
   int		seqdiskswitches;
   int		locdiskswitches;
   int		numlocal;
   int		critreads;
   int		critwrites;
   int		seqreads;
   int		seqwrites;
   int		distavgdiff[10];
   int		distmaxdiff[10];
   double       idlestart;
   double       lastarr;
   double       lastread;
   double       lastwrite;
   int          *blocked;
   int          *aligned;
   int		*lastreq;
   int		*intdist;
   statgen      resptimestats;
   statgen	idlestats;
   statgen      sizestats;
   statgen	readsizestats;
   statgen	writesizestats;
   statgen      intarrstats;
   statgen      readintarrstats;
   statgen      writeintarrstats;
} logorgstat;
```
</details>

<details>
<summary>logorg 结构体</summary>

```c
typedef struct logorg {
  char *name;
   outstand * hashoutstand[HASH_OUTSTAND];
   int    outstandqlen;
   int    opid;
   int    addrbyparts;
   // distribution scheme
   int    maptype;
   // redundancy
   int    reduntype;
   int    numdisks;
   int    actualnumdisks;
   int    arraydisk;
   int    writesync;
   int    copies;
   int    copychoice;
   double rmwpoint;
   int    parityunit;
   int    rottype;
   int    blksperpart;
   int    actualblksperpart;
   int    stripeunit;
   int    sectionunit;
   int    tablestripes;
   tableentry *table;
   int    tablesize;
   int    partsperstripe;
   int    idealno;
   int    reduntoggle;
   int    lastdiskaccessed;
   int    numfull;
   int   *sizes;
   int   *redunsizes;
   int    printlocalitystats;
   int    printblockingstats;
   int    printinterferestats;
   int    printidlestats;
   int    printintarrstats;
   int    printstreakstats;
   int    printstampstats;
   int    printsizestats;
   double stampinterval;
   double stampstart;
   double stampstop;
   FILE * stampfile;
   logorgdev *devs;
   logorgstat stat;
   /* rcohen's additions */
   int    startdev;
   int    enddev;
} logorg;
```
</details>

以及其他一些相关函数的声明。


### disksim_logorg.c

主要关注与 Distribution Scheme 和 Redundancy 有关的代码。

<details>
<summary>logorg_maprequest() 函数</summary>

```c

```
</details>