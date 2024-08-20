---
title: 【学习笔记】NVMe-oF 学习（一）：基础知识
tags:
  - NVMe
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - NVMe
comments: false
cover: false
date: 2024-08-19 21:44:02
---

NVMe 相关基础知识记录。

<!-- more -->

## 相关名词

### PCIe

直连 PCI Express (PCIe) 总线的设计。端对端的互连协议，提供了高速传输带宽的解决方案。点对点的方式连接两个设备。

### NVMe

NVMe 是一种协议，而并非外形规格或接口规范。不同于其他存储协议，NVMe 将 SSD 设备视为内存，而不是硬盘驱动器。NVMe 协议的设计从一开始就以搭配 PCIe 接口使用为目标，因此几乎直接连接到服务器的 CPU 和内存子系统。

NVMe 旨在定义主机软件如何通过 PCI Express (PCIe) 总线与非易失性存储器进行通信，适用于各种 PCIe 固态硬盘 (SSD) 。

### 循环队列

[数据结构：循环队列](https://www.cnblogs.com/chenliyang/p/6554141.html)

### SQ、CQ

* `SQ`: `Submission Queue`，简称 `SQ`。
* `CQ`: `Completion Queue`，简称 `CQ`。

### namespace

[NVMe-namespace](https://www.cnblogs.com/marton/p/12545974.html)

### Host, Target 和 Transport

`client` 端称作 `Host`，处理 `client` 请求的部分称作 `Target` 端（连接物理 `NVMe` 设备），`Host` 和 `Target` 之间使用 `NVMe` `命令交流。Transport` 是连接 `Host` 和 `Target` 的桥梁，可以是 `RDMA` 或者 `FC`。在 `Fabrics` 传输过程中，`NVMe` 命令会被相应的 `Transport` 代码封装（`Capsule`）和解析。

### NVMe Subsystem, NVMe Namespace 和 Port

一个 `Subsystem` 就是一个 `NVMe` 子系统，`Subsystem` 在 `target` 端，`Host` 可以申请连接某个 `target` 的 `Subsystem`。一个 `Port` 代表一个 `Transport` 资源。`Subsystem` 必须和 `Namespace`，`Port` 建立关系，但是他们的联系又是很灵活的：即一个 `Subsystem` 可以包含多个 `Namespace`，一个 `Namespace` 可以加入多个 `Subsystem`，一个 `Port` 可以放入多个 `Subsystem`。




## NVMe 技术概述

[NVMe系列专题之一：NVMe技术概述](https://mp.weixin.qq.com/s?__biz=MzIwNTUxNDgwNg==&mid=2247484348&idx=1&sn=1fd3356c6cd9fee9492dcc7d6eb30345&chksm=972ef2e5a0597bf3190c7e5d7717e25ef3a5d83a1b6d7a20f8c4adb1beafc4372f1d79288dd7&scene=21#wechat_redirect)


## NVMe-oF

NVMe 协议并非局限于在服务器内部连接本地闪存驱动器，它还可通过网络使用。在网络环境内使用时，网络“架构”支持存储和服务器元素之间的任意连接。NVMe-oF 支持组织创建超高性能存储网络，其时延能够媲美直连存储。因而可在服务器之间按需共享快速存储设备。

![NVMe-oF 传输协议](https://ask.qcloudimg.com/http-save/yehe-1419448/e99d89fe79397560d65e0f51d680e2f6.png)

### NVMe over TCP

基于现有的 `IP` 网络，采用 `TCP` 协议传输 `NVMe`，在网络基础设施不变的情况下实现端到端 `NVMe`。

### NVMe over RDMA

`RDMA` 是承载 `NoF` 的原生网络协议，`RDMA` 协议除了 `RoCE` 外还包括 `IB（InfiniBand）`和 `iWARP（Internet Wide Area RDMA Protocol）`。`NVMe over RDMA` 协议比较简单，直接把 `NVMe` 的 `IO` 队列映射到 `RDMA QP（Queue Pair）`连接，通过 `RDMA SEND`，`RDMA WRITE`，`RDMA READ` 三个语义实现 `IO` 交互。