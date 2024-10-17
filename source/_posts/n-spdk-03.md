---
title: 【学习笔记-存储】SPDK（三）：spdk_nvme_perf 代码浅析
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
date: 2024-09-01 19:35:45
---

SPDK 自带的性能测试应用 `spdk_nvme_perf` 的源代码 `perf.c` 的粗浅分析，进一步熟悉 SPDK 用户态驱动的主要工作流程和方式。

<!-- more -->

在看代码的过程中有些不懂的地方在网上查资料后学习了，记录在了 {% post_link n-spdk-04 %} 中。

## 简介

perf 是 SPDK 用来测试 NVMe SSD 性能的工具，最新版本的 SPDK 中 perf 源代码在 `spdk/app/spdk_nvme_perf/` 路径下。perf 主要用来测试 NVMe SSD 的 IOPS，Bandwidth 和 Latency，它既可以测本地的 target，也可以测远端的 target。

## perf 主流程

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/perf-main.png)

---

## perf 资源关系图

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/resource-relationship.png)

---

函数调用栈说明：

* 相同缩进表示同一层的函数执行；不同缩进表示存在函数内调用另一个函数。

* 花括号表示循环体。

* 函数结尾 `;` 表示该函数结束；`:` 表示进入该函数，接下来存在函数调用。

所有函数的调用可以利用 SPDK 的 debug，通过 `-G` 参数开启，不过没有函数的调用层次。

---

## （一）控制流（初始化）

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/dev-stream-init.png)

函数调用栈：

```c
spdk_env_opts_init(&opts);
parse_args(...):
    while {
        getopt_long(..., g_perf_cmdline_opts, ...);
        add_trid(...);
        ... // 创建了 trid 和 trid_entry
    }
spdk_env_init(&opts);
register_workers():
    SPDK_ENV_FOREACH_CORE(i) {
        worker = calloc(...);
    }
    TAILQ_INIT(&worker->ns_ctx);
    TAILQ_INSERT_TAIL(&g_workers, worker, link);
register_controllers():
    TAILQ_FOREACH(trid_entry, &g_trid_list, tailq) {
        spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL):
            probe_ctx = spdk_nvme_probe_async(...): // 注意异步执行
                nvme_driver_init();
                probe_ctx = calloc(...);
                nvme_probe_ctx_init(probe_ctx, ...):
                    TAILQ_INIT(&probe_ctx->init_ctrlrs);
                nvme_probe_internal(probe_ctx, false):
                    nvme_rdma.c: nvme_fabric_ctrlr_scan(probe_ctx, ...):
                        // 该函数调用见后文、构造 ctrlr 函数
            nvme_init_controllers(probe_ctx):
                while (true) { // 注意这里有一个无限 while 循环，意味着 nvme_ctrlr_process_init() 在一直循环执行
                    spdk_nvme_probe_poll_async(probe_ctx): // 注意异步执行
                        TAILQ_FOREACH_SAFE(ctrlr, &probe_ctx->init_ctrlrs, ...) {
                            nvme_ctrlr_poll_internal(ctrlr, probe_ctx):
                                /** 处理 ctrlr 状态转移 **/
                                nvme_ctrlr_process_init(ctrlr):
                                    // 该函数调用见后文、处理状态转移函数
                                TAILQ_REMOVE(&probe_ctx->init_ctrlrs, ctrlr, ...);
                                TAILQ_INSERT_TAIL(&g_nvme_attached_ctrlrs, ctrlr, tailq);
                                perf.c: attach_cb(...):
                                    register_ctrlr(ctrlr, trid_entry):
                                        build_nvme_name(ctrlr_entry->name, ..., ctrlr);
                                        TAILQ_INSERT_TAIL(&g_controllers, ctrlr_entry, ...);
                                        foreach (nsid) {
                                            ns = spdk_nvme_ctrlr_get_ns(ctrlr, nsid);
                                            register_ns(ctrlr, ns);
                                                ... // 进行一系列 io 检查和验证，保证可以正常进行 io 操作
                                                ns_entry->u.nvme.ctrlr = ctrlr;
	                                            ns_entry->u.nvme.ns = ns;
                                                build_nvme_ns_name(ns_entry->name, ..., ctrlr, ...);
                                                TAILQ_INSERT_TAIL(&g_namespaces, entry, link);
                                        }
                        }
                }
    }
```

初始化大体步骤：

* 解析输入参数，初始化 SPDK、DPDK 环境；
* 创建、注册 SPDK 工作线程；
* 通过解析参数得到的 `transport id`、进行**异步设备探测**、通过**状态机**获取设备（控制器、`NS` 等）信息、注册设备（创建 `entry` 通过尾队列维护）等。

注意**异步**以及控制器设备注册时的**状态机**。这一部分也会发送少量请求。

并且刚开始的 `trid` 并不包含任何真正信息，作用就是给程序指明参数提供了有哪些 `trid` 需要去探测。

---

## （二）控制流（初始化 - fabric_ctrlr_scan）

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/dev-stream-fabric-ctrlr-scan.png)

函数调用栈：

```c
nvme_rdma.c: nvme_fabric_ctrlr_scan(probe_ctx, ...):
    spdk_nvme_ctrlr_get_default_ctrlr_opts(&discovery_opts, ...):
        SET_FIELD(...);
    /** discovery_ctrlr 识别过程、没有获得 ns、subnqn 等过程 **/
    discovery_ctrlr = nvme_rdma_ctrlr_construct(&probe_ctx->trid, &discovery_opts, ...):
        ... // 构造 rdma_ctrlr 设置部分字段、建立事件通道等
        nvme_ctrlr_construct(&rctrlr->ctrlr):
            nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_INIT, NVME_TIMEOUT_INFINITE);
            ... // 设置 discovery_ctrlr 部分字段
            TAILQ_INIT(&ctrlr->active_io_qpairs);
            TAILQ_INIT(&ctrlr->active_procs);
            RB_INIT(&ctrlr->ns);
        rctrlr->ctrlr.adminq = nvme_rdma_ctrlr_create_qpair(&rctrlr->ctrlr, ..., async = true):
            rqpair->state = NVME_RDMA_QPAIR_STATE_INVALID; // 创建了 rqpair、设置状态为 INVALID
	        qpair = &rqpair->qpair;
            nvme_qpair_init(qpair, qid, ctrlr, qprio, num_requests, async):
                ... // 设置 qpair 部分字段
                qpair->ctrlr = ctrlr;
	            qpair->trtype = ctrlr->trid.trtype;
	            qpair->async = async;
                STAILQ_INIT(&qpair->free_req);
                for {
                    STAILQ_INSERT_HEAD(&qpair->free_req, req, stailq);
                }
        nvme_ctrlr_add_process(&rctrlr->ctrlr, 0):
            ... // 设置 discovery ctrlr_proc 字段
            TAILQ_INSERT_TAIL(&ctrlr->active_procs, ctrlr_proc, tailq);
    while (discovery_ctrlr->state != NVME_CTRLR_STATE_READY) {
        nvme_ctrlr_process_init(discovery_ctrlr):
            // 该函数调用见后文、处理状态转移函数
    }
    nvme_ctrlr_cmd_identify(discovery_ctrlr, ..., &discovery_ctrlr->cdata, ...);
    nvme_wait_for_completion(discovery_ctrlr->adminq, ...);
    /** io ctrlr 识别过程、完整流程 **/
    nvme_fabric_ctrlr_discover(discovery_ctrlr, probe_ctx): 
        while {
            nvme_fabric_get_discovery_log_page(discovery_ctrlr, buffer, ...):
                spdk_nvme_ctrlr_cmd_get_log_page(...);
                nvme_wait_for_completion(discovery_ctrlr->adminq, status);
            for {
                nvme_fabric_discover_probe(log_page_entry++, probe_ctx, discovery_ctrlr->trid.priority):
                    ... // 通过 log_page_entry 设置 trid
                    nvme_ctrlr_probe(&trid, probe_ctx, NULL):
                        spdk_nvme_ctrlr_get_default_ctrlr_opts(&discovery_opts, ...):
                            SET_FIELD(...);
                        /** 这里构建了 io ctrlr **/
                        ctrlr = nvme_rdma_ctrlr_construct(trid, &opts, devhandle = NULL):
                            rctrlr = spdk_zmalloc(...);
                            ... // 获取 rdma 设备、设备属性等，设置 rctrlr 部分字段
                            nvme_ctrlr_construct(&rctrlr->ctrlr):
                                nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_INIT, NVME_TIMEOUT_INFINITE);
                                ... // 设置 ctrlr 部分字段
                                TAILQ_INIT(&ctrlr->active_io_qpairs);
                                TAILQ_INIT(&ctrlr->active_procs);
                                RB_INIT(&ctrlr->ns);
                            ... // rctrlr 初始化 cm_events 队列
                            // 以下两步在建立 rdma cm 通道，在后面建立 rdma qpairs 连接时用到
                            rctrlr->cm_channel = rdma_create_event_channel();
                            flag = fcntl(rctrlr->cm_channel->fd, F_GETFL);
                            // 创建 admin qpair，qid = 0 为 admin queue
                            rctrlr->ctrlr.adminq = nvme_rdma_ctrlr_create_qpair(&rctrlr->ctrlr, qid = 0, ..., async = true):
                                rqpair->state = NVME_RDMA_QPAIR_STATE_INVALID; // 创建了 rqpair、设置状态为 INVALID
	                            qpair = &rqpair->qpair;
                                nvme_qpair_init(qpair, qid, ctrlr, qprio, num_requests, async):
                                    ... // 设置 qpair 部分字段
                                    qpair->ctrlr = ctrlr;
	                                qpair->trtype = ctrlr->trid.trtype;
	                                qpair->async = async;
                                    STAILQ_INIT(&qpair->free_req);
                                    for {
                                        STAILQ_INSERT_HEAD(&qpair->free_req, req, stailq);
                                    }
                            nvme_ctrlr_add_process(&rctrlr->ctrlr, 0):
                                ... // 设置 ctrlr_proc 字段
                                TAILQ_INSERT_TAIL(&ctrlr->active_procs, ctrlr_proc, tailq);
                                }
                        nvme_qpair_set_state(ctrlr->adminq, NVME_QPAIR_ENABLED);
		                TAILQ_INSERT_TAIL(&probe_ctx->init_ctrlrs, ctrlr, tailq);
        }
```

这部分是扫描创建 ctrlr 核心函数。大体步骤：

* 创建 `discovery_ctrlr` 和其 `Admin QP`，通过不完全状态机，作用是获取 `discovery_log_page`；
* `discovery_log_page` 其中包含了 `ctrlr` 真正有用的信息如 `trid`、`subnqn` 等，用来构建 `io_ctrlr`；
* `io_ctrlr` 经过完全状态机的状态转移，直到 READY 状态，加入到 `init_ctrlrs` 队列中。

注意 `ctrlr` 的状态机，不同的状态执行不同的函数，如识别 NS、获取 Data 等，会通过 `Admin QP` 发送 `identify` 请求。

---

## （三）控制流（初始化 - ctrlr_process_init）

Controller 状态转移过程。

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/dev-stream-ctrlr-process-init.png)

函数调用栈：

```c
 /** This function will be called repeatedly during initialization 
     until the controller is ready. */
nvme_ctrlr_process_init(ctrlr):
    // 状态 CONNECT_ADMINQ: 
    nvme_transport_ctrlr_connect_qpair(ctrlr, ctrlr->adminq):
        nvme_qpair_set_state(qpair, NVME_QPAIR_CONNECTING);
        ... // 与建立 io qpairs 流程类似，见后文
    // 状态 WAIT_FOR_CONNECT_ADMINQ: 
    spdk_nvme_qpair_process_completions(ctrlr->adminq, 0):
        nvme_transport_qpair_process_completions(qpair, max_completions):
            nvme_rdma_qpair_process_completions(qpair, max_completions):
                TODO
    switch (ctrlr->adminq->state) {
        // QPAIR_CONNECTING:
			break;
        // QPAIR_CONNECTED:
        nvme_qpair_set_state(ctrlr->adminq, NVME_QPAIR_ENABLED);
        // QPAIR_ENABLED: 
        nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_READ_VS, NVME_TIMEOUT_INFINITE);
        nvme_qpair_abort_queued_reqs(ctrlr->adminq);
    }
    ...
    // 状态 RESET_ADMIN_QUEUE:
    nvme_transport_qpair_reset(ctrlr->adminq):
        // do nothing
    nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_IDENTIFY, ...);
    // 状态 IDENTIFY:
    nvme_ctrlr_identify(ctrlr):
        nvme_ctrlr_cmd_identify(ctrlr, ..., &ctrlr->cdata, nvme_ctrlr_identify_done, ctrlr);
        nvme_ctrlr_identify_done(ctrlr);
    ...
    // 状态 SET_KEEP_ALIVE_TIMEOUT:
    nvme_ctrlr_set_keep_alive_timeout(ctrlr):
        nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_WAIT_FOR_KEEP_ALIVE_TIMEOUT, ...);
        spdk_nvme_ctrlr_cmd_get_feature(ctrlr, ..., nvme_ctrlr_set_keep_alive_timeout_done, ctrlr);
        nvme_ctrlr_set_keep_alive_timeout_done(ctrlr):
            spdk_nvme_ctrlr_is_discovery(ctrlr); // 判断是否为 dicovery_ctrlr
            if (discovery_ctrlr): nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_READY, NVME_TIMEOUT_INFINITE);
            else if (io_ctrlr): nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_IDENTIFY_IOCS_SPECIFIC, ...);
    // 状态 IDENTIFY_IOCS_SPECIFIC:
    nvme_ctrlr_identify_iocs_specific(ctrlr):
        nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_WAIT_FOR_IDENTIFY_IOCS_SPECIFIC, ...);
        nvme_ctrlr_cmd_identify(ctrlr, ..., ctrlr->cdata_zns, nvme_ctrlr_identify_zns_specific_done, ctrlr);
        nvme_ctrlr_identify_zns_specific_done(ctrlr):
            nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_SET_NUM_QUEUES, ...);
    // 状态 SET_NUM_QUEUES:
    nvme_ctrlr_update_nvmf_ioccsz(ctrlr);
	nvme_ctrlr_set_num_queues(ctrlr):
        nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_WAIT_FOR_SET_NUM_QUEUES, ...);
        nvme_ctrlr_cmd_set_num_queues(ctrlr, ctrlr->opts.num_io_queues, nvme_ctrlr_set_num_queues_done, ctrlr);
        nvme_ctrlr_set_num_queues_done(ctrlr):
            for (i = 1; i <= ctrlr->opts.num_io_queues) {
                // 初始化 free io queue IDs 列表（bit array）
		        // QID 0 是 admin queue，已经 implicitly allocated
                spdk_nvme_ctrlr_free_qid(ctrlr, i);
	        }
            nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_IDENTIFY_ACTIVE_NS, ...);
    // 状态 IDENTIFY_ACTIVE_NS:
    _nvme_ctrlr_identify_active_ns(ctrlr):
        active_ns_ctx = nvme_active_ns_ctx_create(ctrlr, _nvme_active_ns_ctx_deleter); // 绑定 deleter
        nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_WAIT_FOR_IDENTIFY_ACTIVE_NS, ...);
        nvme_ctrlr_identify_active_ns_async(active_ns_ctx): // 异步处理
            nvme_ctrlr_cmd_identify(ctrlr, ..., &active_ns_ctx->new_ns_list[1024 * (active_ns_ctx->page_count - 1)], nvme_ctrlr_identify_active_ns_async_done, active_ns_ctx);
            nvme_ctrlr_identify_active_ns_async_done(active_ns_ctx):
                _nvme_active_ns_ctx_deleter(): // 函数指针
                    nvme_ctrlr_identify_active_ns_swap(ctrlr, ctx->new_ns_list, ...):
                        foreach (active_nsid) {
                            nvme_ctrlr_construct_namespace(ctrlr, nsid):
                                ns = spdk_nvme_ctrlr_get_ns(ctrlr, nsid):
                                    RB_INSERT(nvme_ns_tree, &ctrlr->ns, ns);
                        }
                    nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_IDENTIFY_NS, ...);
    // 状态 IDENTIFY_NS:
    nvme_ctrlr_identify_namespaces(ctrlr):
        ... // 获取 nsid、active_ns、active_ns 绑定 ctrlr
        nvme_ctrlr_identify_ns_async(ns):
            nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_WAIT_FOR_IDENTIFY_NS, ...);
            nvme_ctrlr_cmd_identify(ns->ctrlr, ..., &ns->nsdata, nvme_ctrlr_identify_ns_async_done, ns);
            nvme_ctrlr_identify_ns_async_done(ns):
                nvme_ns_set_identify_data(ns);
                ... // 获取下一个 nsid、active_ns、active_ns 绑定 ctrlr
                if (ns == NULL): nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_IDENTIFY_ID_DESCS, ...);
                else: nvme_ctrlr_identify_ns_async(ns):
                    ... // 循环执行直到 ns == NULL
    // 状态 IDENTIFY_ID_DESCS:
    nvme_ctrlr_identify_id_desc_namespaces(ctrlr);
        ...
   // 状态 IDENTIFY_NS_IOCS_SPECIFIC:
    nvme_ctrlr_identify_namespaces_iocs_specific(ctrlr);
    ...
    // 状态 TRANSPORT_READY:
    nvme_transport_ctrlr_ready(ctrlr); // 没有找到实现函数
    nvme_ctrlr_set_state(ctrlr, NVME_CTRLR_STATE_READY, NVME_TIMEOUT_INFINITE);
    // 状态 NVME_CTRLR_STATE_READY.
    // 状态 WAIT_FOR_XXX 统一处理：（通过 callback 执行转换下一个状态操作）
    spdk_nvme_qpair_process_completions(ctrlr->adminq, 0):
        nvme_transport_qpair_process_completions(qpair, max_completions):
            nvme_rdma_qpair_process_completions(qpair, max_completions):
                TODO
```

不同状态执行不同的函数，函数执行完毕后会手动改变 `ctrlr` 的状态，从而使 `ctrlr` 进入下一个状态。

---

## （四）控制流（任务执行）

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/dev-stream-work-fn.png)

函数调用栈：

```c
/** 创建轮询线程执行任务 **/
pthread_create(&thread_id, NULL, &nvme_poll_ctrlrs, NULL);
// 进入轮询线程：
nvme_poll_ctrlrs():
    while (true) {
        foreach (entry, &g_controllers, link) {
            spdk_nvme_ctrlr_process_admin_completions(entry->ctrlr):
                nvme_ctrlr_keep_alive(ctrlr); // 发送 keep alive 指令
                nvme_io_msg_process(ctrlr): // 处理 ctrlr io 消息
                    if (ctrlr->needs_io_msg_update): nvme_io_msg_ctrlr_update(ctrlr):
                        STAILQ_FOREACH(io_msg_producer, &ctrlr->io_producers, link) {
		                    io_msg_producer->update(ctrlr);
	                    }
                    spdk_nvme_qpair_process_completions(ctrlr->external_io_msgs_qpair, 0):
                        nvme_transport_qpair_process_completions(qpair, max_completions):
                            nvme_rdma_qpair_process_completions(qpair, max_completions):
                                TODO
                    count = spdk_ring_dequeue(ctrlr->external_io_msgs, requests, ...); // 从外部 I/O 消息环中按需排队获取消息，存储在 requests 数组中
                    for (i < count) {
                        spdk_nvme_io_msg *io = requests[i];
                        io->fn(io->ctrlr, io->nsid, io->arg); // 调用 io 消息处理函数
                    }
                spdk_nvme_qpair_process_completions(ctrlr->adminq, 0):
                    nvme_transport_qpair_process_completions(qpair, max_completions):
                        nvme_rdma_qpair_process_completions(qpair, max_completions):
                                TODO
                if (active_proc): nvme_ctrlr_complete_queued_async_events(ctrlr); // 如果当前进程在 ctrlr->active_procs 中，则处理 async_events
        }
    }

// 回到主线程 main()：
associate_workers_with_ns():
    for (max(g_num_namespaces, g_num_workers)) {
        allocate_ns_worker(ns_entry, worker):
            ns_ctx->entry = entry;
            TAILQ_INSERT_TAIL(&worker->ns_ctx, ns_ctx, link); // worker_ns_ctx 加入 worker->ns_ctx 队列中
    }
pthread_barrier_init(&g_worker_sync_barrier, NULL, g_num_workers); // barrier 线程同步，当所有 thread 都执行到 barrier_wait() 后，才继续执行后续代码
TAILQ_FOREACH(worker, &g_workers, link) {
    if (worker->lcore != g_main_core): spdk_env_thread_launch_pinned(worker->lcore, work_fn, worker); // worker thread 一一对应到 core、每个 core 绑定一个 work_fn
}
work_fn(main_worker);
// 每个 core 执行 work_fn
work_fn(worker):
    TAILQ_FOREACH(ns_ctx, &worker->ns_ctx, link) {
        nvme_init_ns_worker_ctx(ns_ctx): // 为每个 ns 分配 io_qpairs
            spdk_nvme_ctrlr_get_default_io_qpair_opts(ns_entry->u.nvme.ctrlr, &opts, sizeof(opts));
            ctrlr_opts = spdk_nvme_ctrlr_get_opts(ns_entry->u.nvme.ctrlr);
            ns_ctx->u.nvme.group = spdk_nvme_poll_group_create(ns_ctx, NULL):  // 创建 nvme_poll_group
                group->ctx = ns_ctx;
	            STAILQ_INIT(&group->tgroups); // tgroup: transport_poll_group
            for (i < ns_ctx->u.nvme.num_all_qpairs) {
                // u.nvme.qpair 是二维指针
                // 1. 创建 io qpair
                ns_ctx->u.nvme.qpair[i] = spdk_nvme_ctrlr_alloc_io_qpair(ns_entry->u.nvme.ctrlr, &opts, ...):
                    qpair = nvme_ctrlr_create_io_qpair(ctrlr, &opts):
                        qid = spdk_nvme_ctrlr_alloc_qid(ctrlr);
                        qpair = nvme_rdma_ctrlr_create_io_qpair(ctrlr, qid, opts): // 函数指针
                            nvme_rdma_ctrlr_create_qpair(ctrlr, qid, ..., opts->async_mode = true):
                                // 与前文 fabric_ctrlr_scan 创建 adminq 类似
                                rqpair->state = NVME_RDMA_QPAIR_STATE_INVALID; // 创建了 rqpair、设置状态为 INVALID
	                            qpair = &rqpair->qpair;
                                nvme_qpair_init(qpair, qid, ctrlr, qprio, num_requests, async):
                                    ... // 设置 qpair 部分字段
                                    qpair->ctrlr = ctrlr;
	                                qpair->trtype = ctrlr->trid.trtype;
	                                qpair->async = async;
                                    STAILQ_INIT(&qpair->free_req);
                                    for {
                                        STAILQ_INSERT_HEAD(&qpair->free_req, req, stailq);
                                    }
                        TAILQ_INSERT_TAIL(&ctrlr->active_io_qpairs, qpair, tailq);
                        nvme_ctrlr_proc_add_io_qpair(qpair): // 与当前 active_proc 关联
                            TAILQ_INSERT_TAIL(&active_proc->allocated_io_qpairs, qpair, per_process_tailq);
		                    qpair->active_proc = active_proc;
                // create_only = true
                // 2. qpair 添加 poll groups
                spdk_nvme_poll_group_add(nvme_poll_group, qpair): // io_qpair 添加到 nvme_poll_group、transport_poll_group
                    while (transport) {
                        tgroup = nvme_transport_poll_group_create(transport): // 创建 transport_poll_group
                            nvme_rdma_poll_group_create():  // 创建 rdma_poll_group
                                STAILQ_INIT(&rdma_poll_group->pollers);
	                            TAILQ_INIT(&rdma_poll_group->connecting_qpairs);
	                            TAILQ_INIT(&rdma_poll_group->active_qpairs);
                                return rdma_poll_group->transport_poll_group;
                            STAILQ_INIT(&group->connected_qpairs);
		                    STAILQ_INIT(&group->disconnected_qpairs);
                        tgroup->group = nvme_poll_group; // transport_poll_group 关联 nvme_poll_group
				        STAILQ_INSERT_TAIL(&nvme_poll_group->tgroups, tgroup, link);
                    }
                    nvme_transport_poll_group_add(tgroup, qpair):
                        qpair->poll_group = tgroup; // qpair 关联 transport_poll_group
                        nvme_rdma_poll_group_add(tgroup, qpair):
                            // do nothing
                            // 这里 rdma_poll_group 没有与 qpair 关联，在后面 rdma_poll_group 与 rdma_qpair 关联
                // 3. 连接 io qpair
                spdk_nvme_ctrlr_connect_io_qpair(entry->u.nvme.ctrlr, qpair):
                    nvme_transport_ctrlr_connect_qpair(ctrlr, qpair):
                        nvme_qpair_set_state(qpair, NVME_QPAIR_CONNECTING); // 这里设置了 qpair 状态为正在连接
                        nvme_rdma_ctrlr_connect_qpair(ctrlr, qpair): // 函数指针
                            // 通过 nvme_qpair、nvme_ctrlr 移动 n 个字节得到
                            rqpair = nvme_rdma_qpair(qpair): // rqpair 为 nvme_rdma_qpair
                                SPDK_CONTAINEROF(qpair, struct nvme_rdma_qpair, qpair):
                                    #define SPDK_CONTAINEROF(ptr, type, member) ((type *)((uintptr_t)ptr - offsetof(type, member))); 
                                    /* offsetof(...): Offset of member MEMBER in a struct of type TYPE. */
                            rctrlr = nvme_rdma_ctrlr(ctrlr); // rctrlr 为 nvme_rdma_ctrlr
                            nvme_parse_addr(&dst_addr, ctrlr->trid.traddr, &port, ...); // 解析 destination_address
                            nvme_parse_addr(&src_addr, ctrlr->opts.src_addr, &src_port, ...); // 解析 source_address
                            rdma_create_id(rctrlr->cm_channel, &rqpair->cm_id, rqpair, RDMA_PS_TCP); // 分配 communication id
                            nvme_rdma_resolve_addr(rqpair, &src_addr, &dst_addr):
                                rdma_set_option(rqpair->cm_id, ...);
                                rdma_resolve_addr(rqpair->cm_id, src_addr, dst_addr, ...);
                                // 开始处理 rdma events
                                nvme_rdma_process_event_start(rqpair, RDMA_CM_EVENT_ADDR_RESOLVED = 0, nvme_rdma_addr_resolved):
                                    rqpair->evt_cb = evt_cb; // evt_cb = nvme_rdma_addr_resolved
                                    ... // 经过一系列操作，最后是怎么执行回调函数 nvme_rdma_addr_resolved()？
                                    /** 这里的 rdma_process_event_start() 调用逻辑见后文 **/
                            rqpair->state = NVME_RDMA_QPAIR_STATE_INITIALIZING; // 注意这里设置了 rqpair 状态，可能用于状态机
                            rgroup = nvme_rdma_poll_group(qpair->poll_group); // rgroup 为 nvme_rdma_poll_group
		                    TAILQ_INSERT_TAIL(&rgroup->connecting_qpairs, rqpair, link_connecting); // rdma_poll_group 与 rdma_qpair 关联
                        nvme_poll_group_connect_qpair(qpair):
                            nvme_transport_poll_group_connect_qpair(qpair):
                                nvme_rdma_poll_group_connect_qpair(qpair):
                                    // do nothing
                                    // 这里 rdma_poll_group 没有连接 rqair
                                    // rqpair 在什么时候从 rgroup->connecting_qpairs 中移除并加入到 rgroup->active_qpairs？
                                    // active_qpairs 是在 submit_request 中才使用，因此作用理解为当 rdma_qpair 有 req 时，加入到 active_qpairs 中
			                    STAILQ_INSERT_TAIL(&tgroup->connected_qpairs, qpair, poll_group_stailq);
                        // qpair->async = true
                // 4. poll group 处理 qpairs 完成事件
                while {
                    /** poll for completions on all qpairs in this poll group **/
                    // 与 检查 io 数据流 类似，见后文
                    spdk_nvme_poll_group_process_completions(group, 0, perf_disconnect_cb):
                        STAILQ_FOREACH(tgroup, &group->tgroups, link) {
                            nvme_transport_poll_group_process_completions(tgroup, ..., disconnected_qpair_cb):
                                nvme_rdma_poll_group_process_completions(tgroup, ..., disconnected_qpair_cb): // 函数指针
                                    TAILQ_FOREACH_SAFE(rqpair, &rgroup->connecting_qpairs, ...) {
                                        nvme_rdma_ctrlr_connect_qpair_poll(rqpair->qpair->ctrlr, rqpair->qpair):
                                            // TODO 这一块没有看懂
                                            // rdma_qpair 状态机
                                            // 根据不同状态执行不同操作、通过 callback 转移状态、直到状态为 NVME_RDMA_QPAIR_STATE_RUNNING 结束
                                            // 在这里可能执行了 rqpair->evt_cb = nvme_rdma_addr_resolved() 函数
                                            // 然后 rdma 解析完 addr 后进行路由
                                            // 按以上思路走
                                            // rqpair->state = NVME_RDMA_QPAIR_STATE_INITIALIZING
                                            ...
                                        TAILQ_REMOVE(&rgroup->connecting_qpairs, rqpair, ...);
                                        // submit queued requests
                                        nvme_qpair_resubmit_requests(rqpair->qpair, rqpair->num_entries):
                                            ... // 与 发送数据流 类似，见后文
                                    }
                                    STAILQ_FOREACH_SAFE(qpair, &tgroup->connected_qpairs, ...) {
                                        nvme_rdma_qpair_process_cm_event(rqpair);
                                            ... // rdma_qpair 状态机
                                    }
                                    STAILQ_FOREACH(poller, &rgroup->pollers, ...) {
                                        nvme_rdma_cq_process_completions(poller->cq, batch_size, poller, NULL, &rdma_completions); // 处理 rdma cq
                                            ... // 与 检查 io 数据流 类似，见后文
                                    }
                        }
		            spdk_nvme_poll_group_all_connected(group):
                        ... // 检查是否都 connected，否则 while 循环继续
                }
            }
    }
    pthread_barrier_wait(&g_worker_sync_barrier); // 线程同步
    TAILQ_FOREACH(ns_ctx, &worker->ns_ctx, link) {
        submit_io(ns_ctx, g_queue_depth); // 发送数据流，见后文
	}
    while (!g_exit) {
        TAILQ_FOREACH(ns_ctx, &worker->ns_ctx, link) {
            ns_ctx->entry->fn_table->check_io(ns_ctx); // 检查 io 数据流，见后文
            ... // 计时、信息统计
        }
        ... // 遍历完一次队列之后就检查时间，如果超过指定的时间就退出，否则继续
    }
    ... // 计时、信息统计、ns_ctx 清理
// 回到主线程 main()：
spdk_env_thread_wait_all();
print_stats(); // 打印信息统计
pthread_barrier_destroy(&g_worker_sync_barrier);
... // cleanup
unregister_trids();
unregister_namespaces();
unregister_controllers();
unregister_workers();
spdk_env_fini();
```

工作任务大体步骤：

* 一般每个线程管理一个 NS；
* 为每个 NS 创建 n 个（假设一个）`io QP`；
* `io QP` 需要与 Target 端 `io QP` 建立对应连接，因此需要建立 RDMA 连接；
* 为 `io QP` 创建轮询组 `poll_group` 以及轮询器 `poller`；
* 准备发送 IO 和检查 IO 完成情况。

---

## （五）控制流（建立 RDMA 层连接）

<!-- 流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/) -->

函数调用栈：

```c
/** 开始解析地址 **/
nvme_rdma_resolve_addr(rqpair, &src_addr, &dst_addr):
    rdma_set_option(rqpair->cm_id, ...);
    rdma_resolve_addr(rqpair->cm_id, src_addr, dst_addr, ...);
    // 开始处理 rdma events
    nvme_rdma_process_event_start(rqpair, RDMA_CM_EVENT_ADDR_RESOLVED = 0, nvme_rdma_addr_resolved):
        // 略
```

RDMA 建立连接部分略。

在这一步中，在 RDMA 建立连接完成后，`io QP` 创建了 rdma_req 池和 rdma_rsp 池，并且 rsps 全部压到 `RQ` 中。

---

## （六）数据流（发送）

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/data-stream-send.png)

函数调用栈：

```c
/* Submit initial I/O for each namespace. */
submit_io(ns_ctx, g_queue_depth):
    while (queue_depth-- > 0) {
		task = allocate_task(ns_ctx, queue_depth):
            nvme_setup_payload(task, pattern = queue_depth % 8 + 1):
                ... // 设置 io 负载
            task->ns_ctx = ns_ctx;
		submit_single_io(task):
            nvme_submit_io(task, task->ns_ctx, task->ns_ctx->entry, offset_in_ios):
                qp_num = ns_ctx->u.nvme.last_qpair++;
                spdk_nvme_ns_cmd_read_with_md(entry->u.nvme.ns, ns_ctx->u.nvme.qpair[qp_num], ..., io_complete, ...): // io_complete() callback
                    payload = NVME_PAYLOAD_CONTIG(buffer, metadata); // 配置 payload
                    req = _nvme_ns_cmd_rw(ns, qpair, &payload, SPDK_NVME_OPC_READ, ...):
                        req = nvme_allocate_request(qpair, payload, ...): // 从请求池中分配 req
                            req = STAILQ_FIRST(&qpair->free_req);
                            STAILQ_REMOVE_HEAD(&qpair->free_req, stailq);
                            NVME_INIT_REQUEST(req, cb_fn, cb_arg, *payload, ...);
                        ... // 配置 req
                        _nvme_ns_cmd_setup_request(ns, req, ...); // 配置 req->cmd
                    nvme_qpair_submit_request(qpair, req): // 请求提交到 qpair
                        _nvme_qpair_submit_request(qpair, req):
                            nvme_transport_qpair_submit_request(qpair, req):
                                // admin queue 不支持将 transport 存储在 ctrlr 或 admin queue 中
                                // io queue 支持
                                if (!nvme_qpair_is_admin_queue(qpair)): qpair->transport->ops.qpair_submit_request(qpair, req);
                                else: transport->ops.qpair_submit_request(qpair, req): // transport 通过 nvme_get_transport(qpair->ctrlr->trid.trstring) 获取
                                    nvme_rdma_qpair_submit_request(qpair, req):
                                        rqpair = nvme_rdma_qpair(qpair);
                                        rdma_req = nvme_rdma_req_get(rqpair);
                                        nvme_rdma_req_init(rqpair, req, rdma_req):
                                            rdma_req->req = req;
	                                        req->cmd.cid = rdma_req->id;
                                            ... // 配置 rdma_req
                                        TAILQ_INSERT_TAIL(&rqpair->outstanding_reqs, rdma_req, link); // 添加到 rqpair->outstanding_reqs 队列中
                                        TAILQ_INSERT_TAIL(&rgroup->active_qpairs, rqpair, link_active); // rqpair 添加到 rgroup->active_qpairs 队列中
                                        // 发送请求排队
                                        spdk_rdma_qp_queue_send_wrs(rqpair->rdma_qp, wr);
                                        // 提交发送请求
                                        nvme_rdma_qpair_submit_sends(rqpair);
                                            // 刷新 rqpair
                                            spdk_rdma_qp_flush_send_wrs(rqpair->rdma_qp, &bad_send_wr):
                                                // 提交发送请求至 send queue
                                                ibv_post_send(spdk_rdma_qp->qp, spdk_rdma_qp->send_wrs.first, bad_wr);
                                                /** 发送请求的过程结束 **/
                                
	}
```

发送 IO 大体步骤：

* 以 IO 任务的形式下发。这个 IO 任务在后续被接收完成后会被复用，直到超过运行时间后回收内存并释放；
* IO 任务可以看作是 `io queue` 的槽，一共会有 `g_queue_depth` 个 IO 任务；
* 在发送前，设置 IO 任务的 io 偏移和随机读写，然后提交任务生成 `nvme_req` 请求；
* `nvme SQ` 的队列长度一般小于 `io queue` 的 `g_queue_depth`，所以会出现 IO 任务排队；
* 通过层层封装、转换，经过 `perf_task -> nvme_req -> rdma_req` 后，通过 RDMA ibverbs 发送请求。

---

## （七）数据流（检查 IO 完成情况）

流程图：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/data-stream-check-io.png)

函数调用栈：

```c
check_io(ns_ctx):
    nvme_check_io(ns_ctx):
        /** poll for completions on all qpairs in this poll group **/
        spdk_nvme_poll_group_process_completions(ns_ctx->u.nvme.group, g_max_completions, perf_disconnect_cb):
            STAILQ_FOREACH(tgroup, &group->tgroups, link) {
                nvme_transport_poll_group_process_completions(tgroup, completions_per_qpair, ...):
                    nvme_rdma_poll_group_process_completions(tgroup, completions_per_qpair, ...):
                        // 1. 处理 cm_events
                        STAILQ_FOREACH_SAFE(qpair, &tgroup->connected_qpairs, ...) {
                            rqpair = nvme_rdma_qpair(qpair);
                            /** event 为空，没有执行
                            nvme_rdma_qpair_process_cm_event(rqpair);
                            */
                        }
                        // 2. poll CQ
                        STAILQ_FOREACH(poller, &rgroup->pollers, link) {
                            while {
                                nvme_rdma_cq_process_completions(poller->cq, batch_size, poller, rdma_qpair = NULL, &rdma_completions):
                                    rc = ibv_poll_cq(cq, batch_size, ibv_wc); // 从 CQ 中取 work completion
                                    for (i < rc) {
                                        rdma_wr = (struct nvme_rdma_wr *)ibv_wc[i].wr_id;
                                        /** 处理 recv */
                                        if (rdma_wr->type == RDMA_WR_TYPE_RECV): nvme_rdma_process_recv_completion(poller, &ibv_wc[i], rdma_wr): // rdma work request 类型为接收
                                            rdma_rsp = SPDK_CONTAINEROF(rdma_wr, struct spdk_nvme_rdma_rsp, rdma_wr);
                                            rqpair = rdma_rsp->rqpair;
                                            rdma_req = &rqpair->rdma_reqs[rdma_rsp->cpl.cid];
                                            rdma_req->completion_flags |= NVME_RDMA_RECV_COMPLETED;
	                                        rdma_req->rdma_rsp = rdma_rsp;
                                            // 这里是如何不进入 if 代码段？
                                            if ((rdma_req->completion_flags & NVME_RDMA_SEND_COMPLETED) == 0) {
                                        		return 0;
	                                        }
                                            nvme_rdma_request_ready(rqpair, rdma_req):
                                                rdma_rsp = rdma_req->rdma_rsp;
                                                recv_wr = rdma_rsp->recv_wr;
                                                recv_wr->next = NULL;
                                                nvme_rdma_req_complete(rdma_req, &rdma_rsp->cpl, true):
                                                    req = rdma_req->req;
                                                    qpair = rdma_req->req->qpair;
                                                    rqpair = nvme_rdma_qpair(qpair);
                                                    TAILQ_REMOVE(&rqpair->outstanding_reqs, rdma_req, link);
                                                    nvme_complete_request(req->cb_fn, req->cb_arg, qpair, req, rsp):
                                                        _nvme_free_request(req, qpair); // 回收 req
                                                        io_complete(cb_arg = perf_task, cpl):
                                                            task_complete(task):
                                                                ... // 计时、信息统计
                                                    nvme_rdma_req_put(rqpair, rdma_req); // 回收 rreq
                                                // 接收请求排队
                                                // 疑问：为啥要提交接收请求？
                                                // 可能是因为 RQ 队列需要随时准备 recv_wr 来随时准备接收？
                                                spdk_rdma_qp_queue_recv_wrs(rqpair->rdma_qp, recv_wr):
                                                    rdma_queue_recv_wrs(&spdk_rdma_qp->recv_wrs, first, &spdk_rdma_qp->stats->recv);
                                            // 提交接收请求
                                            nvme_rdma_qpair_submit_recvs(rqpair):
                                                spdk_rdma_qp_flush_recv_wrs(rqpair->rdma_qp, &bad_recv_wr);
                                                    ibv_post_recv(spdk_rdma_qp->qp, spdk_rdma_qp->recv_wrs.first, bad_wr);
                                        /** 处理 send */
                                        else (rdma_wr->type == RDMA_WR_TYPE_SEND): nvme_rdma_process_send_completion(poller, rdma_qpair, &ibv_wc[i], rdma_wr): // rdma work request 类型为发送
                                            rdma_req = SPDK_CONTAINEROF(rdma_wr, struct spdk_nvme_rdma_req, rdma_wr);
                                            rqpair = nvme_rdma_qpair(rdma_req->req->qpair);
                                            rdma_req->completion_flags |= NVME_RDMA_SEND_COMPLETED;
                                            // ？
                                            if ((rdma_req->completion_flags & NVME_RDMA_RECV_COMPLETED) == 0) {
                                        		return 0;
	                                        }
                                            /** 以下代码段没有执行
                                            nvme_rdma_request_ready(rqpair, rdma_req):
                                                rdma_rsp = rdma_req->rdma_rsp;
                                                recv_wr = rdma_rsp->recv_wr;
                                                nvme_rdma_req_complete(rdma_req, &rdma_rsp->cpl, true):
                                                    qpair = rdma_req->req->qpair;
                                                    rqpair = nvme_rdma_qpair(qpair);
                                                    TAILQ_REMOVE(&rqpair->outstanding_reqs, rdma_req, link);
                                                    nvme_complete_request(req->cb_fn, req->cb_arg, qpair, req, rsp):
                                                        _nvme_free_request(req, qpair); // 回收 req
                                                        io_complete(cb_arg = perf_task, cpl):
                                                            task_complete(task):
                                                                ... // 计时、信息统计
                                                    nvme_rdma_req_put(rqpair, rdma_req); // 回收 rreq
                                                // 接收请求排队
                                                // 疑问：为啥要接收请求？
                                                spdk_rdma_qp_queue_recv_wrs(rqpair->rdma_qp, recv_wr):
                                                    rdma_queue_recv_wrs(&spdk_rdma_qp->recv_wrs, first, &spdk_rdma_qp->stats->recv);
                                            // 提交接收请求
                                            nvme_rdma_qpair_submit_recvs(rqpair):
                                                spdk_rdma_qp_flush_recv_wrs(rqpair->rdma_qp, &bad_recv_wr);
                                                    ibv_post_recv(spdk_rdma_qp->qp, spdk_rdma_qp->recv_wrs.first, bad_wr);
                                            **/
                                    }

                            }
                        }
                        // 3. 活跃 QP 提交 send/recv 请求
                        TAILQ_FOREACH_SAFE(rqpair, &group->active_qpairs, link_active, tmp_rqpair) {
		                    nvme_rdma_qpair_process_submits(group, rqpair):
                                // 提交 send_wrs 和 recv_wrs
                                nvme_rdma_qpair_submit_sends(rqpair);
                                nvme_rdma_qpair_submit_recvs(rqpair);
                                // 重新提交 queued 请求
                                if (rqpair->num_completions > 0): nvme_qpair_resubmit_requests(qpair, rqpair->num_completions);
                        }
            }
``` 

检查 IO 完成情况大体步骤：

* `CQ` 收到 `CQE` 后，通过结构体偏移得到 `WQE`，判断是 `SEND` 还是 `RECV` 请求的完成消息；
* 只有当 `WQE` 指向的 `rdma_req` 的 `completion_flags` 为 `SEND_COMPLETED & RECV_COMPLETED` 即发送和接收都完成了，才判断为 IO 任务完成；
* 通过 callback 得到原始 IO 任务；
* 记录时间，统计信息；
* IO 任务被复用，重新设置 io 偏移量以及随机读写，重复发送过程。

---