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

本身就是。

---

## 同一个 IO 请求复制多份分发到 QP



---

