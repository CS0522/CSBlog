---
title: 【学习笔记-存储】SPDK（四）：补充知识记录
tags:
  - 存储
  - SPDK
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2024-09-18 19:23:39
---

记录在看 perf 代码中学习到的 NVMe、NVMe-oF、RDMA、SPDK 相关知识点。

<!-- more -->

## NVMe

> 参考[介绍 NVMe](https://blog.csdn.net/BGONE/article/details/123467570)

### NVMe Specifications

* NVM Express Base Specification 定义了主机软件通过各种基于内存的传输和基于消息的传输与非易失性内存子系统通信的协议。

* NVM Express Management Interface Specification 为所有 NVM Express 子系统定义了一个可选的管理接口。

* NVM Express I/O Command Set Specification 定义了扩展 NVM Express Base Specification 的数据结构、features、log pages、commands 和 status 值。

* NVM Express Transport Specification 定义了 NVMe 协议（包括控制器属性）与特定传输的绑定。

### NVMe 控制器类型

所有控制器都实现了一个 Admin Submission Queue 和一个 Admin Completion Queue。Identify Controller data structure 中的 Controller Type (CNTRLTYPE) 字段指示控制器的类型。(cdata->cntrltype)

* **I/O controllers**

实现 I/O 队列的控制器，旨在用于访问非易失性内存存储介质。

* **Discovery controllers**

公开**允许主机检索 Discovery Log Page 的 capabilities 的控制器**。Discovery controller 不实现 I/O 队列、I/O 命令或公开命名空间。

* **Administrative controllers**

公开允许主机管理 NVM 子系统的 capabilities 的控制器。Administrative 控制器不实现 I/O 队列，不提供对与非易失性内存存储介质上的用户数据相关的数据或元数据的访问，也不支持命名空间（即，从来没有任何 active NSID）附加到 Administrative 控制器。

### Identify data structure

* Identify Controller data structures

能够通过 identify 命令检索的所有控制器数据结构：Identify Controller data structure （即，CNS 01h）和每个 I/O Command Set 特定的 Identify Controller data structure（即，CNS 06h）。

* Identify Namespace data structures

能够通过 Identify 命令检索的所有命名空间数据结构：Identify Namespace data structure（即，CNS 00h）、I/O Command Set Independent Identify Namespace data structure（即，CNS 08h）和每个 I/O Command Set 特定的 Identify Namespace data structures（即 05h）。

### SQ、CQ

* 提交队列（Submission Queues，SQ）

用于发送命令到 NVMe 驱动器。主机将命令放入提交队列后，驱动器会从中取出并执行这些命令。

* 完成队列（Completion Queues，CQ）

用于接收来自 NVMe 驱动器的命令完成通知。当命令在驱动器中完成时，相关的信息会被放入完成队列。

NVMe 中这两个队列为 qpair。每个命名空间可以有多个队列（通常最大为 64K），这使得 NVMe 在处理高并发读写请求时非常高效。

---

### SGL/SGE

[SGL 和 SGE](https://zhuanlan.zhihu.com/p/55142568)

## RDMA

### SQ、RQ、CQ

* 发送队列（Send Queue，SQ）

用于存放待发送的消息或数据请求。当应用程序准备发送数据时，会将其放入发送队列。

* 接收队列（Receive Queue，RQ）

用于存放接收数据的请求。应用程序在接收数据前，会将接收请求放入接收队列。

* 完成队列（Completion Queue，CQ）

用于存放已完成操作的信息。当一个发送或接收操作完成时，相关的信息会被放入完成队列，应用程序可以通过检查完成队列来了解操作的状态。

RDMA 中 SQ 和 RQ 为 qpair，CQ 不属于 qpair。

### 通信机制

参考 [基本操作类型 Send/Recv、Write、Read 机制和通信过程](https://blog.csdn.net/lianghuaju/article/details/140240461)

参考 [初识 RDMA 技术](https://zhuanlan.zhihu.com/p/649468433)

RDMA 分为 `基于 Memory` 和 `基于 Messaging` 两种操作：

* 基于 Memory

    * RDMA Read：从远程主机读取部分内存。调用者指定远程虚拟地址，像本地内存地址一样用来拷贝。在执行 RDMA 读操作之前，远程主机必须提供适当的权限来访问它的内存。一旦权限设置完成， RDMA 读操作就可以在对远程主机没有任何通知的条件下执行。不管是 RDMA 读还是 RDMA 写，远程主机都不会意识到操作正在执行（除了权限和相关资源的准备操作）。

    * RDMA Write：与 RDMA Read 类似，只是数据写到远端主机中。RDMA 写操作在执行时不通知远程主机。然而带即时数的 RDMA 写操作会将即时数通知给远程主机。

    * RDMA Atomic：包括原子取、原子加、原子比较和原子交换，属于 RDMA 原子操作的扩展。

* 基于 Messaging

    * RDMA Send：发送操作允许把数据发送到远程 QP 的接收队列里。接收端必须已经事先注册好了用来接收数据的缓冲区。发送者无法控制数据在远程主机中的放置位置。可选择是否使用即时数，一个 4 位的即时数可以和数据缓冲一起被传送。这个即时数发送到接收端是作为接收的通知，不包含在数据缓冲之中。
    
    * RDMA Receive：这是与发送操作相对应的操作。接收主机被告知接收到数据缓冲，还可能附带一个即时数。接收端应用程序负责接收缓冲区的维护和发布。

通信过程见参考文章。

## NVMe-oF

### 