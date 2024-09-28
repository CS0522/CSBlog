---
title: 【学习笔记-存储】SPDK（五）：实现监控逻辑和功能测试
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
date: 2024-09-25 10:04:41
---

修改 perf 中的监控功能，进行功能测试，分析存在的问题。

<!-- more -->

## 环境准备

* Ubuntu 22.04 虚拟机 x4
    * **dev1，作为 Host 端，ip：192.168.246.129**
    * **dev2，作为 Target1，ip：192.168.246.130**
    * **dev3，作为 Target2，ip：192.168.246.131**
    * **dev4，作为 Target3，ip：192.168.246.132**
* 8 块虚拟硬盘：
    * SATA x4：用于安装 Ubuntu
    * NVMe x4：用于绑定 spdk


## perf 同时测试多个 Target、设置指定数目 QP

建立环境的操作见 {% post_link n-spdk-02 %}。

Host 端：

```bash
./build/bin/spdk_nvme_perf -r 'trtype:rdma adrfam:IPv4 traddr:192.168.246.130 trsvcid:4420' -r 'trtype:rdma adrfam:IPv4 traddr:192.168.246.131 trsvcid:4420' -r 'trtype:rdma adrfam:IPv4 traddr:192.168.246.132 trsvcid:4420' -q 256 -o 4096 -w randrw -M 50 -t 5 -P 1 -G -LL -l --transport-stats

# 一些参数含义
# -r, --transport <fmt> Transport ID for local PCIe NVMe or NVMeoF
# 对于同时测试多块盘，只需要添加多个 -r 指定设备地址即可
# -q, --io-depth <val> io depth
# -o, --io-size <val> io size in bytes
# -w, --io-pattern <pattern> io pattern type, must be one of: 
# (read, write, randread, randwrite, rw, randrw)
# -M, --rwmixread <0-100> rwmixread (100 for reads, 0 for writes)
# -t, --time <sec> time in seconds
# -P, --num-qpairs <val> number of io queues per namespace. default: 1
# -L, --enable-sw-latency-tracking enable latency tracking via sw, -LL for detailed histogram, default: disabled
# -l, --enable-ssd-latency-tracking enable latency tracking via ssd (if supported), default: disabled
# -G, --enable-debug enable debug logging
# --transport-stats dump transport statistics

# 更多参数含义可以见 --help 或源代码中的 usage() 函数
```

---

## 每个 QP 映射一个 Target

本身就是，每个 NS 建立 1 个 QP，而每个 Target 只有 1 个 NS。

---

## 同一个 IO 请求复制多份分发到 QP

### 如何复制 IO？

初步思路：

* 为 perf_task 添加索引序号 index 字段；

* 由 main_worker 创建 g_queue_depth 个 tasks，每个 task 中关联的 ns_ctx 指针暂时指向 main_worker 中的 ns_ctx；

* 这些 tasks 保存在 g_tasks 指针数组中，并为所有 tasks 添加索引序号，这样所有 worker_thread 都可以获得相同的 task；

* 在分发 task 时，需要注意深拷贝一份新的 task 然后修改关联的 ns_ctx 指针，否则可能会造成各个线程之间的指针互相影响。


### 如何控制多线程能够同时下发 IO req？

**如果是有 n 个线程，> n 个 NS，其中存在 m 个线程关联了 > 1 个 NS，这 m 个线程中的多个 NS 通过 while 循环遍历发送，如何确保同时？**

这里一定需要 worker_thread 和 core 1 对 1？NVMe 架构图好像确实需要 core 和 ns 绑定。

* 在分发 tasks 时，每分发一个，就 barrier 同步一次，确保每个 IO 都是尽可能同时发送。


### TODO 如何知道 nvme_req、rdma_req、rdma_rsp 等属于哪一个 IO？

请求之间转换关系图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-05/req-transform-relationship.png)


疑问：

* wc->wr_id 字段是如何赋值的？

* send_wr->imm_data、wc->imm_data 如何使用？

* rsp->cpl 字段是如何赋值的？


### TODO req 完成后对应的 rsp 的接收问题

把同一个 IO 分成了 3 份，在 Target 处理完毕后，肯定会发送 rsp，Host 端每个 RQ 都会收到一份相同 IO 的 rsp，因此会对 rsp 处理 3 次 recv_completion 执行 3 次 task->complete。

---

## 当前待解决问题

### 3 副本情况下，recv_completion 处理过程中出现段错误

TODO

可能是回收释放内存的过程中出现了问题，待排查。

由于 1 个 task 复制了多份，可能存在多个指针指向同一片内存空间，被释放多次的问题。

也可能指针访问越界。