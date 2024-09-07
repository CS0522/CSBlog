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

## 简介

perf 是 SPDK 用来测试 NVMe SSD 性能的工具，最新版本的 SPDK 中 perf 源代码在 `spdk/app/spdk_nvme_perf/` 路径下。perf 主要用来测试 NVMe SSD 的 IOPS，Bandwidth 和 Latency，它既可以测本地的 target，也可以测远端的 target。

## perf 的 main() 函数主流程

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/perf-main.png)

---

## perf 的宏定义、全局变量、定义的结构体

<!-- TODO -->

---

## （一）环境初始化

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/perf-01-env-init.png)

### 初始化 opts 参数默认值（控制流）

函数调用栈：

```
--> lib/env_dpdk/init.c: spdk_env_opts_init(&opts);
```

这部分的功能比较简单，用于初始化 `spdk_env_opts` 结构体的默认选项，包括共享内存标识符、内存大小、主核心、内存通道数、基本虚拟地址等。

```c
void
spdk_env_opts_init(struct spdk_env_opts *opts)
{
	if (!opts) {
		return;
	}

memset(opts, 0, sizeof(*opts));

	opts->name = SPDK_ENV_DPDK_DEFAULT_NAME;
	opts->core_mask = SPDK_ENV_DPDK_DEFAULT_CORE_MASK;
	opts->shm_id = SPDK_ENV_DPDK_DEFAULT_SHM_ID;
	opts->mem_size = SPDK_ENV_DPDK_DEFAULT_MEM_SIZE;
	opts->main_core = SPDK_ENV_DPDK_DEFAULT_MAIN_CORE;
	opts->mem_channel = SPDK_ENV_DPDK_DEFAULT_MEM_CHANNEL;
	opts->base_virtaddr = SPDK_ENV_DPDK_DEFAULT_BASE_VIRTADDR;
}
```

`spdk_env_opts` 结构体有以下几个域，其他域的初始化赋值应该在后续执行函数中：

```c
struct spdk_env_opts {
	const char		*name;                          // 通常用于标识当前运行的程序或实例。
	const char		*core_mask;                     // 指定哪个 CPU 核心可以用于运行 SPDK 任务，使用 CPU 核心掩码表示。
	const char		*lcore_map;                     // 逻辑核心映射，描述了各个逻辑核心的映射关系，提供细粒度核心调度控制。
	int			    shm_id;                         // 用于在多个进程之间共享数据。
	int			    mem_channel;                    // 指定要使用的内存通道的数量，根据系统的内存架构优化性能。
	int			    main_core;                      // 根据系统的内存架构优化性能。
	int			    mem_size;                       // SPDK 将为其使用分配的内存大小。
	bool			no_pci;                         // 是否禁用 PCI 设备支持。是则不初始化 PCI 设备。
	bool			hugepage_single_segments;       // 是否使用单段大页进行内存分配。
	bool			unlink_hugepage;                // 是否在程序退出时卸载大页面。
	bool			no_huge;                        // 是否禁用使用大页内存。
	size_t			num_pci_addr;                   // PCI 地址的数量，通常用于描述需要管理或使用的 PCI 设备数量。
	const char		*hugedir;                       // 用于指定大页文件的存储位置。
	struct spdk_pci_addr	*pci_blocked;           // 用于描述哪些 PCI 设备在 SPDK 运行时不允许使用。
	struct spdk_pci_addr	*pci_allowed;           // 指定可以使用的 PCI 设备列表。
	const char		*iova_mode;                     
	uint64_t		base_virtaddr;                  // 基础虚拟地址，用于 SPDK 从该地址开始分配内存。

    /** Opaque context for use of the env implementation. */
    void			*env_context;                   // 环境实现的上下文，通常用于存储环境特定的信息。
    const char		*vf_token;                      // 虚拟功能的标识符，指定虚拟功能设备。
};
```

在这个函数执行完毕后，`main()` 中初始化了 `opts->name` 以及 `opts->pci_allowed` 的值：

```c
opts.name = "perf";
opts.pci_allowed = g_allowed_pci_addr;
```

### 解析 args（控制流）

函数调用栈：

```
--> perf.c: parse_args(argc, argv, &opts);
----> dpdk/lib/eal/<windows/linux/freebsd/...>/getopt.c: getopt_long(argc, argv, PERF_GETOPT_SHORT, g_perf_cmdline_opts, &long_idx);
----> perf.c: add_trid(optarg);
------> nvme.c: spdk_nvme_transport_id_parse(trid, trid_str);
--------> nvme.c: spdk_nvme_transport_id_parse_trtype(&trid->trtype, val)
------> TAILQ_INSERT_TAIL(&g_trid_list, trid_entry, tailq);
```

进入 `parse_args()` 函数中，其中通过 `while ((op = getopt_long(argc, argv, PERF_GETOPT_SHORT, g_perf_cmdline_opts, &long_idx)) != -1)`，也就是 `getopt_long()` 函数循环读取参数、为 `opts` 初始化赋值，其中，`g_perf_cmdline_opts` 为全局定义的 `option` 结构体的数组，`struct option g_perf_cmdline_opts[] = {option1, option2, ...}`。

以 {% post_link n-spdk-02 %} 中的 `spdk_nvme_perf` 测试示例为例，调用的参数为：`./build/bin/spdk_nvme_perf -r 'trtype:rdma adrfam:IPv4 traddr:192.168.159.142 trsvcid:4420' -q 256 -o 4096 -w randread -t 100`，其中的指定参数为 `-r`，`-q`，`-o`，`-w`，`-t`：

```c
static const struct option g_perf_cmdline_opts[] = {
...,
// 传输层协议
#define PERF_TRANSPORT	'r'
	{"transport",			required_argument,	NULL, PERF_TRANSPORT},

// io 队列深度。给定时间内，可以同时发出的 io 请求的数量
#define PERF_IO_DEPTH	'q'
	{"io-depth",			required_argument,	NULL, PERF_IO_DEPTH},

// io 请求数据大小
#define PERF_IO_SIZE	'o'
	{"io-size",			required_argument,	NULL, PERF_IO_SIZE},

// 顺序读/写、随机读/写等工作负载
#define PERF_IO_PATTERN	'w'
	{"io-pattern",			required_argument,	NULL, PERF_IO_PATTERN},

// 测试时间
#define PERF_TIME	't'
	{"time",			required_argument,	NULL, PERF_TIME},
}
```

以设置 `trtype` 属性为 `RDMA` 为例子。

与 `rdma` 字样相关的是 `PERF_TRANSPORT`，在 while 循环中找到处理 `PERF_TRANSPORT` 的函数：

```c
case PERF_TRANSPORT:
			if (add_trid(optarg)) {
				usage(argv[0]);
				return 1;
			}
			break;
```

进入 `add_trid()` 函数：

```c
static int
add_trid(const char *trid_str)
{
    // 表示 NVMe 传输标识符条目 (trid) 的信息s
	struct trid_entry *trid_entry;
    // trtype 是 spdk_nvme_transport_id 结构体中的域
	struct spdk_nvme_transport_id *trid;
	char *ns;
	char *hostnqn;

	trid_entry = calloc(1, sizeof(*trid_entry));
	if (trid_entry == NULL) {
		return -1;
	}

	trid = &trid_entry->trid;
	trid->trtype = SPDK_NVME_TRANSPORT_PCIE;
	snprintf(trid->subnqn, sizeof(trid->subnqn), "%s", SPDK_NVMF_DISCOVERY_NQN);

    // 解析 trid
	if (spdk_nvme_transport_id_parse(trid, trid_str) != 0) {
		fprintf(stderr, "Invalid transport ID format '%s'\n", trid_str);
		free(trid_entry);
		return 1;
	}
    ...

    TAILQ_INSERT_TAIL(&g_trid_list, trid_entry, tailq);
    return 0;
}
```

进入 `spdk_nvme_transport_id_parse(trid, trid_str)` 函数：

```c
int
spdk_nvme_transport_id_parse(struct spdk_nvme_transport_id *trid, const char *str)
{
	...

	if (strcasecmp(key, "trtype") == 0) {
		if (spdk_nvme_transport_id_populate_trstring(trid, val) != 0) {
			SPDK_ERRLOG("invalid transport '%s'\n", val);
			return -EINVAL;
		}
		if (spdk_nvme_transport_id_parse_trtype(&trid->trtype, val) != 0) {
			SPDK_ERRLOG("Unknown trtype '%s'\n", val);
			return -EINVAL;
		}
    }

    ...
}
```

进入 `spdk_nvme_transport_id_parse_trtype(&trid->trtype, val)` 函数：

```c
int
spdk_nvme_transport_id_parse_trtype(enum spdk_nvme_transport_type *trtype, const char *str)
{
	if (trtype == NULL || str == NULL) {
		return -EINVAL;
	}

	if (strcasecmp(str, "PCIe") == 0) {
		*trtype = SPDK_NVME_TRANSPORT_PCIE;
	} else if (strcasecmp(str, "RDMA") == 0) {
		*trtype = SPDK_NVME_TRANSPORT_RDMA;
	} else if (strcasecmp(str, "FC") == 0) {
		*trtype = SPDK_NVME_TRANSPORT_FC;
	} else if (strcasecmp(str, "TCP") == 0) {
		*trtype = SPDK_NVME_TRANSPORT_TCP;
	} else if (strcasecmp(str, "VFIOUSER") == 0) {
		*trtype = SPDK_NVME_TRANSPORT_VFIOUSER;
	} else {
		*trtype = SPDK_NVME_TRANSPORT_CUSTOM;
	}
	return 0;
}
```

最终设置了 `transport type` 为 `RDMA`。

到此设置 `trtype` 结束，其他参数初始化过程类似。

### 初始化运行环境（控制流）

函数调用栈：

```
--> lib/env_dpdk/init.c: spdk_env_init(&opts);
----> openssl/ssl.h: OPENSSL_init_ssl(OPENSSL_INIT_LOAD_CONFIG, settings);
----> lib/env_dpdk/init.c: build_eal_cmdline(opts);
------> lib/env_dpdk/init.c: push_arg(args, opts->..);
----> lib/env_dpdk/init.c: rte_eal_init(g_eal_cmdline_argcount, dpdk_args);
------> TODO
----> lib/env_dpdk/init.c: spdk_env_dpdk_post_init(...);
------> lib/env_dpdk/pic.c: pci_env_init();
--------> TODO
------> lib/env_dpdk/memory.c: mem_map_init(...);
------> lib/env_dpdk/memory.c: vtophys_init();
```

初始化 spdk 和 dpdk 环境。

进入 `spdk_env_init(&opts)` 函数中，其中代码流程大致如下：

```c
char **g_eal_cmdline = NULL;
int g_eal_cmdline_argcount = 0;
// 没有初始化过
bool g_external_init = true;

int spdk_env_init(const struct spdk_env_opts *opts)
{
    // dpdk 参数
	char **dpdk_args = NULL;
	char *args_print = NULL, *args_tmp = NULL;
	OPENSSL_INIT_SETTINGS *settings;
	int i, rc;
    // FreeBSD always uses legacy memory model
	bool legacy_mem;

    // 这里 g_external_init 初始化默认为 true
    // 如果已经初始化过，则 g_external_init == false；
    // 没有初始化过，则 g_external_init == true
	if (g_external_init == false) {
        // 重新初始化 pci
        pci_env_reinit();
		return 0;
	}

    // 开始初始化 spdk

    // openssl 安全套接字
	settings = OPENSSL_INIT_new();
    // 初始化 openssl
	rc = OPENSSL_init_ssl(OPENSSL_INIT_LOAD_CONFIG, settings);
	OPENSSL_INIT_free(settings);

    // 通过 opts 中的选项构建 eal
    // eal：环境抽象层，负责为应用间接访问底层的资源
	rc = build_eal_cmdline(opts);

	dpdk_args = calloc(g_eal_cmdline_argcount, sizeof(char *));
    // 将 g_eal_cmdline 的内容复制到 dpdk_args 中
	memcpy(dpdk_args, g_eal_cmdline, sizeof(char *) * g_eal_cmdline_argcount);

    // 初始化 dpdk，使用之前准备的参数
	rc = rte_eal_init(g_eal_cmdline_argcount, dpdk_args);

	// 设置 legacy_mem 为 true/false

    // 初始化后的相关处理
	rc = spdk_env_dpdk_post_init(legacy_mem);
    // 如果成功，将 g_external_init 设置为 false
    // 如果已经初始化过，则 g_external_init == false
	if (rc == 0) {
		g_external_init = false;
	}

	return rc;
}
```

进入到 `build_eal_cmdline(opts)` 函数中，其函数的主要作用就是构建 EAL 命令行参数用于初始化 dpdk，其中调用 `push_arg()` 将命令行参数处理后添加到 `args` 链表尾部，然后赋值给全局的 `g_eal_cmdline`。

EAL 是 dpdk 的环境抽象层，负责为应用间接访问底层的资源。

```c
static int
build_eal_cmdline(const struct spdk_env_opts *opts)
{
	int argcount = 0;
	char **args;
	bool no_huge;

	args = NULL;
	no_huge = opts->no_huge || (opts->env_context && strstr(opts->env_context, "--no-huge") != NULL);

	/* set the program name */
	args = push_arg(args, &argcount, _sprintf_alloc("%s", opts->name));
	if (args == NULL) {
		return -1;
	}
    ...

    g_eal_cmdline = args;
	g_eal_cmdline_argcount = argcount;
	return argcount;
}
```

上面的函数执行完后，将 `g_eal_cmdline` 的内容复制到 `dpdk_args` 中，然后执行 dpdk 的初始化操作 `rte_eal_init(g_eal_cmdline_argcount, dpdk_args)`，这个函数还没细看，看起来应该是初始化 dpdk 的组件，然后创建和加载 threads 并绑核操作。

TODO

最后是 dpdk 初始化完成后的后续处理操作函数 `spdk_env_dpdk_post_init(legacy_mem)`，进入该函数中：

```c
int spdk_env_dpdk_post_init(bool legacy_mem)
{
	int rc;

	rc = pci_env_init();
	if (rc < 0) {
		SPDK_ERRLOG("pci_env_init() failed\n");
		return rc;
	}

	rc = mem_map_init(legacy_mem);
	if (rc < 0) {
		SPDK_ERRLOG("Failed to allocate mem_map\n");
		return rc;
	}

	rc = vtophys_init();
	if (rc < 0) {
		SPDK_ERRLOG("Failed to initialize vtophys\n");
		return rc;
	}

	return 0;
}
```

其中执行了 `pci_env_init()`、`mem_map_init(legacy_mem)`、`vtophys_init()` 等 dpdk 的初始化函数，其中涉及到 pci 驱动注册等细节还没有看，不过不在这篇浅析的范围内。

TODO

最后如果初始化成功，将 `g_external_init` 置为 `false`。对于全局变量 `g_external_init`，如果 spdk 环境已经初始化过，则 `g_external_init = false`；没有初始化过，则 `g_external_init = true`。
	
```c
...
if (rc == 0) {
	g_external_init = false;
}

return rc;
```

到此 `spdk_env_init(&opts)` 的执行结束。

---

## （二）设备发现和注册

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/perf-02-register.png)

### 注册 worker 线程（控制流）

函数调用栈：

```
--> perf.c: register_workers();
----> SPDK_ENV_FOREACH_CORE(i) 
{ 
------> TAILQ_INIT(&worker->ns_ctx);
------> TAILQ_INSERT_TAIL(&g_workers, worker, link);
}
```

进入到 `register_workers()` 函数中，主要作用就是创建线程，为 `ns_worker_ctx` 初始化尾队列，然后将 worker 添加到全局 g_workers 尾队列中：

```c
struct worker_thread {
	TAILQ_HEAD(, ns_worker_ctx)	ns_ctx;
	TAILQ_ENTRY(worker_thread)	link;
	unsigned			lcore;
};

static int
register_workers(void)
{
	uint32_t i;
	struct worker_thread *worker;

	SPDK_ENV_FOREACH_CORE(i) {
		worker = calloc(1, sizeof(*worker));
		if (worker == NULL) {
			fprintf(stderr, "Unable to allocate worker\n");
			return -1;
		}

        // 为 ns_worker_ctx 初始化一个尾队列
        /**尾队列是一种双向链表，支持快速的插入和删除操作。它的主要特点包括：
         * 插入/删除效率: 在队列的头部或尾部插入或删除元素都很高效，因为你只需更新指针。
         * 指针管理: 每个队列头（即 TAILQ_HEAD）结构包含一个指向第一个元素的指针和一个指向最后一个元素的指针的指针。
         * 这允许快速访问队列的两端。
         */
		TAILQ_INIT(&worker->ns_ctx);
		worker->lcore = i;
        // 添加到尾队列
		TAILQ_INSERT_TAIL(&g_workers, worker, link);
		g_num_workers++;
	}

	return 0;
}
```

注册 worker 线程工作结束。


### 探测并初始化 NVMe controllers（控制流）

函数调用栈：

```
--> perf.c: register_controllers();
----> TAILQ_FOREACH(trid_entry, &g_trid_list, tailq)
{
------> nvme.c: spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL);
--------> nvme.c: spdk_nvme_probe_async(trid, cb_ctx, probe_cb, attach_cb, remove_cb);
----------> nvme.c: nvme_driver_init();
------------> lib/env_dpdk/env.c: spdk_memzone_reserve(SPDK_NVME_DRIVER_NAME, ...);
------------> nvme.c: nvme_robust_mutex_init_shared(&g_spdk_nvme_driver->lock);
------------> lib/env_dpdk/pci_event.c: spdk_pci_event_listen();
------------> TAILQ_INIT(&g_spdk_nvme_driver->shared_attached_ctrlrs);
----------> nvme.c: nvme_probe_ctx_init(probe_ctx, trid, cb_ctx, probe_cb, attach_cb, remove_cb);
------------> TAILQ_INIT(&probe_ctx->init_ctrlrs);
----------> nvme.c: nvme_probe_internal(probe_ctx, false);
------------> nvme_transport.c: nvme_transport_ctrlr_scan(probe_ctx, direct_connect);
--------------> nvme_transport.c: nvme_get_transport(probe_ctx->trid.trstring);
--------------> nvme_rdma.c --> nvme_fabric.c: nvme_fabric_ctrlr_scan(probe_ctx, direct_connect);
----------------> TODO
--------> nvme.c: nvme_init_controllers(probe_ctx);
----------> nvme.c: spdk_nvme_probe_poll_async(probe_ctx);
------------> TAILQ_FOREACH_SAFE(ctrlr, &probe_ctx->init_ctrlrs, tailq, ctrlr_tmp)
{
--------------> nvme.c: nvme_ctrlr_poll_internal(ctrlr, probe_ctx);
----------------> nvme_ctrlr.c: nvme_ctrlr_process_init(ctrlr);
------------------> TODO
----------------> STAILQ_INIT(&ctrlr->io_producers);
----------------> TAILQ_REMOVE(&probe_ctx->init_ctrlrs, ctrlr, tailq);
----------------> TAILQ_INSERT_TAIL(&g_nvme_attached_ctrlrs, ctrlr, tailq);
----------------> perf.c: attach_cb(probe_ctx->cb_ctx, &ctrlr->trid, ctrlr, &ctrlr->opts); (callback)
------------------> perf.c: register_ctrlr(ctrlr, trid_entry);
--------------------> perf.c: build_nvme_name(entry->name, sizeof(entry->name), ctrlr);
--------------------> TAILQ_INSERT_TAIL(&g_controllers, entry, link);
--------------------> perf.c: register_ns(ctrlr, ns);
----------------------> TODO
}
}
```

流程简单总结：

TODO

具体流程：

`register_controllers()` 的作用实际是遍历 `g_trid_list`，为每个 `trid` 循环遍历所有检测到的控制器并为每个控制器调用实际注册函数 `register_ctrlr()` 进行注册。进入到该函数中：

```c
struct trid_entry {
	struct spdk_nvme_transport_id	trid;
	uint16_t			nsid;
	char				hostnqn[SPDK_NVMF_NQN_MAX_LEN + 1];
	TAILQ_ENTRY(trid_entry)		tailq;
};

static int
register_controllers(void)
{
    // 包含 transport 相关的信息和功能接口
	struct trid_entry *trid_entry;

	printf("Initializing NVMe Controllers\n");

	if (g_vmd && spdk_vmd_init()) {
		fprintf(stderr, "Failed to initialize VMD."
			" Some NVMe devices can be unavailable.\n");
	}

    // 对每个 trid 进行设备发现
    // 处理探测的回调函数 (probe_cb)、附加设备的回调函数 (attach_cb) 
	TAILQ_FOREACH(trid_entry, &g_trid_list, tailq) {
        // spdk 设备探测
		if (spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL) != 0) {
			fprintf(stderr, "spdk_nvme_probe() failed for transport address '%s'\n",
				trid_entry->trid.traddr);
			return -1;
		}
	}

	return 0;
}
```

对前面已经初始化好的 `g_trid_list` 尾队列进行遍历，也就是对每个 transport id 进行设备探测，其中调用到 `spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL)` 函数，这个函数的作用就是进行 NVMe 设备探测，进入该函数中：

```c
/**参数解释：
 * trid：传入的 transport ID（transport identifier），用于指定要探测的特定 NVMe 设备的类型和地址。
 * cb_ctx：一个上下文指针，通常用于在回调函数中传递特定的上下文信息。
 * probe_cb：探测回调函数。当探测到一个新的 NVMe 设备时，这个回调会被调用，通常用于执行设备的初始化或资源分配。
 * attach_cb：attach 回调函数，当设备成功 attached 时调用。这通常会涉及设备的配置和准备使其可用。
 * remove_cb：移除回调函数，用于处理设备被移除的情况。这使得程序能够清理或释放与设备相关的资源。
 */
int
spdk_nvme_probe(const struct spdk_nvme_transport_id *trid, void *cb_ctx,
		spdk_nvme_probe_cb probe_cb, spdk_nvme_attach_cb attach_cb,
		spdk_nvme_remove_cb remove_cb)
{
	struct spdk_nvme_transport_id trid_pcie;
	struct spdk_nvme_probe_ctx *probe_ctx;

	...

    // 异步探测，返回 probe context 探测上下文信息
	probe_ctx = spdk_nvme_probe_async(trid, cb_ctx, probe_cb,
					  attach_cb, remove_cb);

	/*
	 * Keep going even if one or more nvme_attach() calls failed,
	 *  but maintain the value of rc to signal errors when we return.
	 */
    // 根据 probe context 探测上下文信息，
    // 初始化并 attach 所有 controllers
    // attach 所有 controllers 后返回 0
	return nvme_init_controllers(probe_ctx);
}
```

接着按顺序先执行 `spdk_nvme_probe_async(trid, cb_ctx, probe_cb, attach_cb, remove_cb);` 函数。这个函数的作用是异步地在系统中查找并识别可用的 NVMe 设备，**然后返回探测后的上下文信息**。进入该函数中：

```c
// probe_ctx 的结构体
struct spdk_nvme_probe_ctx {
	struct spdk_nvme_transport_id		trid;
	void					*cb_ctx;
	spdk_nvme_probe_cb			probe_cb;
	spdk_nvme_attach_cb			attach_cb;
	spdk_nvme_remove_cb			remove_cb;
	TAILQ_HEAD(, spdk_nvme_ctrlr)		init_ctrlrs;
};

/**用于异步地在系统中查找并识别可用的 NVMe 设备
 * trid：传入的 transport ID（transport identifier），用于指定要探测的特定 NVMe 设备的类型和地址。
 * cb_ctx：一个上下文指针，通常用于在回调函数中传递特定的上下文信息。
 * probe_cb：探测回调函数。当探测到一个新的 NVMe 设备时，这个回调会被调用，通常用于执行设备的初始化或资源分配。
 * attach_cb：attach 回调函数，当设备成功 attached 时调用。这通常会涉及设备的配置和准备使其可用。
 * remove_cb：移除回调函数，用于处理设备被移除的情况。这使得程序能够清理或释放与设备相关的资源。
 */
struct spdk_nvme_probe_ctx *
spdk_nvme_probe_async(const struct spdk_nvme_transport_id *trid,
		      void *cb_ctx,
		      spdk_nvme_probe_cb probe_cb,
		      spdk_nvme_attach_cb attach_cb,
		      spdk_nvme_remove_cb remove_cb)
{
	int rc;
	struct spdk_nvme_probe_ctx *probe_ctx;

	rc = nvme_driver_init();

	probe_ctx = calloc(1, sizeof(*probe_ctx));

	nvme_probe_ctx_init(probe_ctx, trid, cb_ctx, probe_cb, attach_cb, remove_cb);
    // direct_connect: false
	rc = nvme_probe_internal(probe_ctx, false);

	return probe_ctx;
}
```

首先执行 `nvme_driver_init()` 函数，进入该函数中：

```c
// TODO
```

然后执行 `nvme_probe_ctx_init(probe_ctx, trid, cb_ctx, probe_cb, attach_cb, remove_cb)` 函数，该函数对 `probe_ctx` 这个结构体指针进行赋值，可以看作是连接 `probe_cb`、`attach_cb` 等回调函数：

```c
static void
nvme_probe_ctx_init(struct spdk_nvme_probe_ctx *probe_ctx,
		    const struct spdk_nvme_transport_id *trid,
		    void *cb_ctx,
		    spdk_nvme_probe_cb probe_cb,
		    spdk_nvme_attach_cb attach_cb,
		    spdk_nvme_remove_cb remove_cb)
{
	probe_ctx->trid = *trid;
	probe_ctx->cb_ctx = cb_ctx;
	probe_ctx->probe_cb = probe_cb;
	probe_ctx->attach_cb = attach_cb;
	probe_ctx->remove_cb = remove_cb;
	TAILQ_INIT(&probe_ctx->init_ctrlrs);
}
```

最后执行 `nvme_probe_internal(probe_ctx, false)` 函数，`nvme_probe_internal()` 是 NVMe controllers 探测过程中的核心部分，负责 scan controllers 等。函数利用了 `g_spdk_nvme_driver` 域内的 mutex 以确保线程安全。进入该函数中：

```c
/* This function must only be called while holding g_spdk_nvme_driver->lock */
static int
nvme_probe_internal(struct spdk_nvme_probe_ctx *probe_ctx,
		    bool direct_connect)
{
	int rc;
	struct spdk_nvme_ctrlr *ctrlr, *ctrlr_tmp;

    ...

	nvme_robust_mutex_lock(&g_spdk_nvme_driver->lock);

	rc = nvme_transport_ctrlr_scan(probe_ctx, direct_connect);
	if (rc != 0) {
		SPDK_ERRLOG("NVMe ctrlr scan failed\n");
		...
	}

	/*
	 * Probe controllers on the shared_attached_ctrlrs list
	 */
    // probe_ctx->trid.trtype 值为 RDMA，不执行 if 中代码块
	if (!spdk_process_is_primary() && (probe_ctx->trid.trtype == SPDK_NVME_TRANSPORT_PCIE)) {
        ...
	}

	nvme_robust_mutex_unlock(&g_spdk_nvme_driver->lock);

	return 0;
}
```

进入到 `nvme_transport_ctrlr_scan(probe_ctx, direct_connect);` 函数中：

TODO

最后 `nvme_probe_internal(probe_ctx, direct_connect)` 函数执行结束，返回 `spdk_nvme_probe_async(trid, cb_ctx, probe_cb, attach_cb, remove_cb)` 函数，`spdk_nvme_probe_async()` 也执行完毕，向上返回 `probe_ctx` 探测上下文信息，此时函数调用返回到上一级的 `spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL)` 函数，执行最后的 `nvme_init_controllers(probe_ctx)` 函数，根据上面存储的 `probe_ctx` 上下文，进行 nvme controllers 初始化。进入到 `nvme_init_controllers(probe_ctx)` 函数中：

```c
static int
nvme_init_controllers(struct spdk_nvme_probe_ctx *probe_ctx)
{
	int rc = 0;

	while (true) {
        // 轮询探测，直到返回值不是 -EAGAIN，然后退出循环
		rc = spdk_nvme_probe_poll_async(probe_ctx);
		if (rc != -EAGAIN) {
			return rc;
		}
	}

	return rc;
}
```

执行轮询探测 `spdk_nvme_probe_poll_async(probe_ctx)` 函数，用于异步轮询并初始化 NVMe 控制器。当所有控制器全部初始化完成后，即 `init_ctrlrs` 尾队列为空，会更新 `g_spdk_nvme_driver` 驱动状态 `initialized = true`，并释放资源；否则返回 `-EAGAIN`，继续执行循环轮询。进入到该函数中：

```c
/**
 * 用于异步轮询并初始化 NVMe 控制器。当所有控制器全部初始化完成后，会更新 
 * g_spdk_nvme_driver 驱动状态 initialized = true，并释放资源；
 * 否则返回 -EAGAIN，继续执行循环轮询。
 */
int
spdk_nvme_probe_poll_async(struct spdk_nvme_probe_ctx *probe_ctx)
{
	struct spdk_nvme_ctrlr *ctrlr, *ctrlr_tmp;

    ...

	TAILQ_FOREACH_SAFE(ctrlr, &probe_ctx->init_ctrlrs, tailq, ctrlr_tmp) {
        // 轮询 Controllers 并 attach
		nvme_ctrlr_poll_internal(ctrlr, probe_ctx);
	}

    // 如果控制器链表为空，表示所有控制器都已成功初始化。
	if (TAILQ_EMPTY(&probe_ctx->init_ctrlrs)) {
		nvme_robust_mutex_lock(&g_spdk_nvme_driver->lock);
        // 更新 g_spdk_nvme_driver 驱动状态
        // g_spdk_nvme_driver 在这里初始化标识被设置为 true
		g_spdk_nvme_driver->initialized = true;
		nvme_robust_mutex_unlock(&g_spdk_nvme_driver->lock);
		free(probe_ctx);
		return 0;
	}

	return -EAGAIN;
}
```

在这个函数中，会轮询所有在 `init_ctrlrs` 尾队列中的所有待初始化的控制器，并执行 `nvme_ctrlr_poll_internal(ctrlr, probe_ctx)` 函数，该函数也是轮询探测部分中的核心函数，因此进入该函数中，并传入了待初始化的控制器以及 `probe_ctx` 上下文信息：

```c
static void
nvme_ctrlr_poll_internal(struct spdk_nvme_ctrlr *ctrlr,
			 struct spdk_nvme_probe_ctx *probe_ctx)
{
	int rc = 0;

	rc = nvme_ctrlr_process_init(ctrlr);

	if (rc) {
		/* Controller failed to initialize. */
        ...
	}

	if (ctrlr->state != NVME_CTRLR_STATE_READY) {
		return;
	}

    // 单链表尾队列
    // io_producers 用于管理 io 请求的线程并发处理，
    // 记录控制器上有多少生产者在提交 io 请求
	STAILQ_INIT(&ctrlr->io_producers);

	/*
	 * Controller has been initialized.
	 *  Move it to the attached_ctrlrs list.
	 */
    // 初始化完成后从 init_ctrlrs 队列中移除
    // 添加到 attachec_ctrlrs 队列中
	TAILQ_REMOVE(&probe_ctx->init_ctrlrs, ctrlr, tailq);

	nvme_robust_mutex_lock(&g_spdk_nvme_driver->lock);
	if (nvme_ctrlr_shared(ctrlr)) {
		TAILQ_INSERT_TAIL(&g_spdk_nvme_driver->shared_attached_ctrlrs, ctrlr, tailq);
	} else {
		TAILQ_INSERT_TAIL(&g_nvme_attached_ctrlrs, ctrlr, tailq);
	}

	/*
	 * Increase the ref count before calling attach_cb() as the user may
	 * call nvme_detach() immediately.
	 */
	nvme_ctrlr_proc_get_ref(ctrlr);
	nvme_robust_mutex_unlock(&g_spdk_nvme_driver->lock);

    // 执行 attach_cb 回调函数
	if (probe_ctx->attach_cb) {
		probe_ctx->attach_cb(probe_ctx->cb_ctx, &ctrlr->trid, ctrlr, &ctrlr->opts);
	}
}
```

首先执行了 `nvme_ctrlr_process_init(ctrlr)` 函数，

TODO

中间执行了一些控制器在不同队列中进行切换的操作，最后执行 `attach_cb` 回调函数，`attach_cb(probe_ctx->cb_ctx, &ctrlr->trid, ctrlr, &ctrlr->opts)` 位于 `perf.c` 中，进入该函数：

```c
// attach_cb 回调函数
static void
attach_cb(void *cb_ctx, const struct spdk_nvme_transport_id *trid,
	  struct spdk_nvme_ctrlr *ctrlr, const struct spdk_nvme_ctrlr_opts *opts)
{
	struct trid_entry	*trid_entry = cb_ctx;
	struct spdk_pci_addr	pci_addr;
	struct spdk_pci_device	*pci_dev;
	struct spdk_pci_id	pci_id;

    // rdma 等执行 if 内代码块
	if (trid->trtype != SPDK_NVME_TRANSPORT_PCIE) {
		printf("Attached to NVMe over Fabrics controller at %s:%s: %s\n",
		       trid->traddr, trid->trsvcid,
		       trid->subnqn);
	} 
    // pcie 执行 else 内代码块
    else {
		...
	}

	register_ctrlr(ctrlr, trid_entry);
}
```

执行 `register_ctrlr(ctrlr, trid_entry)` 函数，初始化并注册一个新的 NVMe 控制器，并根据传入的 trid_entry 配置信息处理相关的命名空间 ns。

```c
struct trid_entry {
	struct spdk_nvme_transport_id	trid;
	uint16_t			nsid;
	char				hostnqn[SPDK_NVMF_NQN_MAX_LEN + 1];
	TAILQ_ENTRY(trid_entry)		tailq;
};

static void
register_ctrlr(struct spdk_nvme_ctrlr *ctrlr, struct trid_entry *trid_entry)
{
	struct spdk_nvme_ns *ns;
	struct ctrlr_entry *entry = malloc(sizeof(struct ctrlr_entry));
	uint32_t nsid;

    ...

    // 为控制器生成一个 name
	build_nvme_name(entry->name, sizeof(entry->name), ctrlr);

	entry->ctrlr = ctrlr;
	entry->trtype = trid_entry->trid.trtype;
    // perf.c: register_controllers() 函数中，
    // 实际上到这一步，才将 controller 加入到全局 g_controllers 队列中
	TAILQ_INSERT_TAIL(&g_controllers, entry, link);

    // 延迟跟踪功能，通过 parse_args() 解析命令行参数
    ...

    // 如果 nsid 为 0，则会遍历该控制器的所有活动命名空间，
    // 并逐一调用 register_ns() 进行注册。
	if (trid_entry->nsid == 0) {
		for (nsid = spdk_nvme_ctrlr_get_first_active_ns(ctrlr);
		     nsid != 0; nsid = spdk_nvme_ctrlr_get_next_active_ns(ctrlr, nsid)) {
			ns = spdk_nvme_ctrlr_get_ns(ctrlr, nsid);
			if (ns == NULL) {
				continue;
			}
			register_ns(ctrlr, ns);
		}
	} else {
		ns = spdk_nvme_ctrlr_get_ns(ctrlr, trid_entry->nsid);

        // 注册命名空间
		register_ns(ctrlr, ns);
	}
}
```

注册命名空间，调用 `register_ns(ctrlr, ns)` 函数，进入该函数：

TODO

```c

```



最后 `nvme_ctrlr_poll_internal(ctrlr, probe_ctx)` 函数执行结束，返回上一级。

`spdk_nvme_probe_poll_async(probe_ctx)` 函数执行结束，返回上一级。

`nvme_init_controllers(probe_ctx)` 函数执行结束，返回上一级。

`spdk_nvme_probe(&trid_entry->trid, trid_entry, probe_cb, attach_cb, NULL)` 函数执行结束，返回到 `perf.c` 中的 `register_controllers()` 函数中。

`register_controllers()` 函数执行结束，`perf.c` 中的 `main()` 函数继续执行，在进行下一步前，进行了一些全局资源的检查：

```c
if (g_warn) {
	printf("WARNING: Some requested NVMe devices were skipped\n");
}

if (g_num_namespaces == 0) {
	fprintf(stderr, "No valid NVMe controllers or AIO or URING devices found\n");
	goto cleanup;
}

if (g_num_workers > 1 && g_quiet_count > 1) {
	fprintf(stderr, "Error message rate-limiting enabled across multiple threads.\n");
	fprintf(stderr, "Error suppression count may not be exact.\n");
}
```

---

## （三）多线程的任务执行

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/)

### 