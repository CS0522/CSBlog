---
title: 【学习笔记】SPDK（五）：实现监控逻辑和功能测试
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
    * **dev2，作为 Target0，ip：192.168.246.130**
    * **dev3，作为 Target1，ip：192.168.246.131**
    * **dev4，作为 Target2，ip：192.168.246.132**
* 8 块虚拟硬盘：
    * SATA x4：用于安装 Ubuntu
    * NVMe x4：用于绑定 SPDK


---

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

本身就是，每个 `NS` 建立 1 个 `QP`，而每个 `Target` 只有 1 个 `NS`。

---

## perf 当前 IO 任务下发、回收逻辑

简洁版：

```c
/*** 下发 ***/
foreach (ns_ctx) {
    while (queue_depth-- > 0) {
        ... // 创建一个 task
        ... // 发送 task
            ... // 经过多层调用、每层都会对 task 某些字段进行配置
            submit_single_io(task): // 这一层设置了 random offset 和 random read/write
            nvme_submit_io(task):
                ... // 通过 io_complete callback 关联 req 与 task
                ... // 创建 nvme_req、rdma_req、ibv_send_wr 等
                qp_send_wrs(send_wr); // 最终发送 send_wr
    }
}

/*** 回收 ***/
...
process_recv_completion(...):
    ... // rdma_req 打上 RECV_COMPLETED 标记
    ... // 因为是同一个 rdma_req，所以在发送成功后已经有 SEND_COMPLETED 标记
    ... // 如果同时有 RECV 和 SEND 完成标记，则会进行下一步
    io_complete(task):
        task_complete(task):
            if (ns_ctx->is_draining): free(task); // 如果超过时间，则开始释放 IO 任务
            else: submit_single_io(task):
                ... // 没有超过时间，则会重新利用 task、设置 random offset 和 random read/write
```

重要的点在于**如果没有超过时间，接收到的 req 对应的 task 会被重新利用，buffer 的地址不变、但 random offset 和 random read/write 的值会发生随机改变。**

---

## 各层的 req 与 task 关联以及互相关联

请求之间转换关系图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-05/req-transform-relationship.png)


疑问：

* `wc->wr_id` 字段是如何赋值的？

* `send_wr->imm_data`、`wc->imm_data` 如何使用？

* `rsp->cpl` 字段是如何赋值的？

---

## 修改 perf IO 任务逻辑 - 初版

需要考虑兼容单 `worker` 和多 `worker` 的情形。

### 同一个 IO 任务复制多份

* 为 `perf_task` 添加索引序号 `io_id` 字段；

* 由 `main_worker` 创建 `g_queue_depth` 个 `tasks`，每个 `task` 中关联的 `ns_ctx` 指针暂时指向某个 `ns_ctx`；

* 这些 `tasks` 保存在 `g_tasks` 指针数组中，这样所有 `worker_thread (ns)` 都可以通过复制得到相同的 `task`；

* 在分发 `task` 时，需要深拷贝一份新的 `task` 然后修改关联的 `ns_ctx`、`iovs` 等指针。

### task 一致性

由 `perf` 下发和回收逻辑看出，下发到多个 `qpairs` 的 `task` 之间，会发生改变的主要是 `task->io_id`、`task->iovs (buffer)`、`task->md_iovs`、`offset_in_ios`、`task->is_read` 等字段和变量。因此控制当 `task->io_id` 相等时，其他的字段和变量也对应一致就可以满足 `task` 复制的要求。

不需要要求多个 `IO` 同时发送，因此可以采取任务队列的类似思路。

* 由 `main_worker`：
    * 创建 `g_queue_depth` 个 `g_tasks`；
    * `g_random_num = 2 * g_queue_depth`；
    * 创建 `g_random_num` 长度的 `g_offset_in_ios` 数组；
    * 创建 `g_random_num` 长度的 `g_is_read` 数组。

* 提前通过 `srand(time(NULL))` 和 `rand()` 初始化好 `g_offset_in_ios` 和 `g_is_read`；

* 每提交一个 `task` 时，从 `g_offset_in_ios` 和 `g_is_read` 中取出下标为 `task->io_id % (q_random_num)` 的随机值，这样就可以保证提交的下标相同的 `task` 的随机偏移量和 `r/w` 是一致的。

这样的思路理论上可以保证提交的下标相同的 `task` 的随机偏移量和 `r/w` 是一致的；而 `task` 的其他字段则在创建 `g_tasks` 时就已经配置好。

### 接收请求后重新利用 task 并提交

原 `task` 回收逻辑中，是将之前发送过的 `task` 重新利用，其中的字段都不变，仅修改 `offset_in_ios` 和 `is_read` 的值，然后创建新的 `req`。

多副本的情况下，需要保证 `offset_in_ios` 和 `is_read` 一致，同时能够跟踪到 `task_io_id`，所以修改的思路为：

* `task->io_id` 在每次收到后都 `+= g_queue_depth`，这样不会导致 `task->io_id` 的重复（同一个 `ns` 提交多次 `task->io_id = n` 的请求）；

* 重新利用 `task->io_id` 并提交时，会进入到 `nvme_submit_io()` 函数中，在这个函数里为 `offset_in_ios` 和 `is_read` 进行了赋值，修改为直接获取下标为 `task->io_id % (g_random_num)` 的已经保存在数组中的随机值。

### 多副本 IO 同步

暂未考虑。

### 多个 task 内存回收

在 `cleanup` 阶段统一清理 `g_tasks` 数组。

### 修改后 IO 任务下发、回收逻辑

简洁版图示：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-05/rep-01.png)

---

## 修改 perf IO 任务逻辑 - 优化后

假设在单 `worker` 情形。

### 同一个 IO 任务复制多份

* 分主从副本 `main_task` 和 `rep_task`，其中 `main_task` 维护了一个副本队列，包括自己在内的所有副本；

* 每个副本都有指向主副本 `main_task` 的指针字段；

* 每个 `worker` 都可以感知到所有 `ns_ctx`；

* `worker` 遍历其所有 `ns_ctx`，第一个对应的副本为主副本；当创建主副本后，其他的 `ns_ctx` 对应的副本都为从副本。


### task 一致性

* 当主副本设置完 io 偏移量以及随机读写后，遍历其副本队列，同步给所有从副本；

* 然后执行提交 IO 任务的逻辑。

### 接收请求后重新利用 task 并提交

* `task->io_id` 在每次收到后都 `+= g_queue_depth`，这样不会导致 `task->io_id` 的重复（同一个 `ns` 提交多次 `task->io_id = n` 的请求）；

* 重新利用 `task->io_id` 并提交时，会进入到 `nvme_submit_io()` 函数中，在这个函数里为 `offset_in_ios` 和 `is_read` 进行了重新随机赋值，之后主副本再次同步给所有从副本。

### 多副本 IO 同步

存在两次同步：

1. 下发同一个 IO 任务时，提交主副本。在主副本提交前遍历其副本队列，将他们都提交到相应 NS 的队列；

2. 任务完成后，主副本的计数器满足要求才代表该 IO 任务完成。即当所有副本均完成后，该 IO 任务才算完成，然后才对主副本重新设置。

因此可能会造成某个 IO 任务的某个副本还没有完成，从而导致下一个 IO 无法下发的情况，但理论上这与实际不符。

### 多个 task 内存回收

* 达到运行时间后，回收主副本；

* 每个副本指向的 `iovs` 内存地址只释放一次，而每个副本都要被回收释放。

### 修改后 IO 任务下发、回收逻辑

简洁版图示：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-05/rep-02.png)

---