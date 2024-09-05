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

## （一）perf 环境初始化部分

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/perf-01-env-init.png)

### 初始化 opts 参数默认值

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

### 解析 args

```
--> perf.c: parse_args(argc, argv, &opts);
----> dpdk/lib/eal/windows/getopt.c: getopt_long(argc, argv, PERF_GETOPT_SHORT, g_perf_cmdline_opts, &long_idx);
----> perf.c: add_trid(optarg);
------> nvme.c: spdk_nvme_transport_id_parse(trid, trid_str);
--------> nvme.c: spdk_nvme_transport_id_parse_trtype(&trid->trtype, val)
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
    // 表示一个 NVMe 传输标识符条目 (trid) 的信息s
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

### 初始化运行环境

```
--> lib/env_dpdk/init.c: spdk_env_init(&opts);
----> openssl/ssl.h: OPENSSL_init_ssl(OPENSSL_INIT_LOAD_CONFIG, settings);
----> lib/env_dpdk/init.c: build_eal_cmdline(opts);
------> lib/env_dpdk/init.c: push_arg(args, opts->..);
----> lib/env_dpdk/init.c: rte_eal_init(g_eal_cmdline_argcount, dpdk_args);
----> lib/env_dpdk/init.c: spdk_env_dpdk_post_init(...);
------> lib/env_dpdk/pic.c: pci_env_init();
------> lib/env_dpdk/memory.c: mem_map_init(...);
------> lib/env_dpdk/memory.c: vtophys_init();
```

初始化 spdk 和 dpdk 环境。

进入 `spdk_env_init(&opts)` 函数中，其中代码流程大致如下：

```c
char **g_eal_cmdline = NULL;
int g_eal_cmdline_argcount = 0;
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

## （二）perf 设备发现和注册

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-spdk-03/)

### 