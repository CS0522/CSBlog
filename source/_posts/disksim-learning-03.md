---
title: DiskSim 学习（三）：程序流程
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
date: 2023-11-06 15:17:11
---

DiskSim 的 main 函数

<!-- more -->

## disksim_main.c

DiskSim 的 main 函数位于 `disksim-4.0/src/disksim_main.c` 文件里。代码如下：

```c
int main (int argc, char **argv)
{
	int len;

#ifndef _WIN32
	setlinebuf(stdout);
	setlinebuf(stderr);
#endif

	if(argc == 2) {
		disksim_restore_from_checkpoint (argv[1]);
	} 
	else {
		// disksim.c:120
        disksim = calloc(1, sizeof(struct disksim));
        // 初始化结构体
        disksim_initialize_disksim_structure(disksim);
        // 设置 disksim
        // disksim.c:911
        disksim_setup_disksim (argc, argv);
	}
	disksim_run_simulation ();
	disksim_cleanup_and_printstats ();
	exit(0);
}
```

主要分为以下几个步骤。

### 给全局变量 disksim 分配内存

代码中 `disksim` 变量没有声明而直接使用，因为他是一个全局变量，在 `disksim.c:120` 有其初始化的语句：

```c
// disksim_global.h:342
disksim_t *disksim = NULL;
```

### 初始化 disksim 结构体

调用 `disksim_initialize_disksim_structure(disksim)` 函数，函数定义在 `disksim.c:1117`：

```c
int disksim_initialize_disksim_structure (struct disksim *disksim)
{
   disksim->closedthinktime = 0.0;
   warmuptime = 0.0;	/* gets remapped to disksim->warmuptime */
   simtime = 0.0;	/* gets remapped to disksim->warmuptime */
   disksim->lastphystime = 0.0;
   disksim->checkpoint_interval = 0.0;

   return 0;
}
```

### 建立 disksim

调用 `disksim_setup_disksim (argc, argv)`，主要是对 disksim 做一些初始化操作。
大致功能：设置对齐方式（大端对齐还是小端对齐）、设置输出文件、设置 trace 的格式、
设置输入的 trace 数据的文件、判断是否启用 synthgen、设置重写的参数、根据配置文件设置参数、
iosim_info 的初始化，最后为开始模拟磁盘做准备。后面继续学习这个函数。

### 开始模拟

这里开始模拟磁盘的操作，
通过 `disksim_run_simulation ()` 函数来开始模拟；没有传入任何参数，可以看出模拟的是全局的 `disksim`。

模拟结束后释放内存并打印输出结果。