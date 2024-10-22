---
title: 【学习笔记-存储】SPDK（六）：CloudLab 脚本自动化部署 SPDK
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
date: 2024-10-17 10:22:09
---

记录在 CloudLab 上通过 sh 脚本自动化部署 SPDK with latency_test。

<!-- more -->

## 1. 脚本介绍

| 名称 | 使用位置 | 作用 |
| :---: | :---: | :---: |
| setup_all_nodes.sh | 本地 | 整合所有功能脚本；<br/>遍历所有节点并配置 |
| setup_rdma.sh | 远程 | 配置 RDMA 环境 |
| setup_spdk_with_latency_test.sh | 远程 | 搭建添加了 latency_test 的 SPDK 环境 |
| configure_host.sh | 远程 | 配置 Host 端 |
| configure_target.sh | 远程 | 配置 Target 端 |
| run_perf_test.sh | 本地 / 远程 | 运行 perf、perf_rep |
| get_run_output.sh | 本地 / 远程 | 获取所有节点的输出结果 |


## 2. 脚本使用方法
### 2.1. 工作路径

由于 CloudLab 服务器要求，需要持久化存储的文件要保存在 `/opt`或 `/usr/local`文件夹下，这里选择 `/opt`。

+ 远程机器的 SPDK 根目录：`/opt/Workspace/spdk-24.05.x`
+ 远程机器的输出文件目录：`/opt/Workspace/output`
+ 本地机器的脚本存放目录：`./setup`
+ 本地机器运行脚本后的输出文件目录：
    - `./setup/run_output`，存放获取远程机器的输出文件；
    - `./setup/setup_output`，存放 setup 过程中的文件和输出，包括 `nodes_local_ip.txt`、`setup_all_nodes.log`等。

### 2.2. 脚本参数
#### setup_all_nodes.sh

`./setup/setup_all_nodes.sh <is_mlnx> <is_100g> <cloudlab_username> <node_num> [hostnames...]`

+ `is_mlnx`

值为 0 或 1。

是否为 Mellanox 网卡。CloudLab 环境上满足实验要求的 NIC 均为  Mellanox 网卡。

+ `is_100g`

值为 0 或 1。

网速是否为 100 Gbps，是为 1，mtu = 9000；否为 0，mtu = 1500；通过 mtu 的大小来获取目标网口的 IP。

+ `cloudlab_username`

用户名：CS0522，需要大写。

+ `node_num`

节点个数，1 Host + 多 Target。

+ `hostnames...`

节点的主机名。值如 `amd263.utah.cloudlab.us`，多值中间空格分隔，在 CloudLab 实验界面获取。其中输入的第一个节点默认为 Host，其他为 Target。

#### setup_rdma.sh

`./setup/setup_rdma.sh <is_mlnx> <is_100g>`

+ `is_mlnx`

值为 0 或 1。

是否为 Mellanox 网卡。CloudLab 环境上满足实验要求的 NIC 均为  Mellanox 网卡。

+ `is_100g`

值为 0 或 1。

网速是否为 100 Gbps，是为 1，mtu = 9000；否为 0，mtu = 1500；通过 mtu 的大小来获取目标网口的 IP。

#### setup_spdk_with_latency_test.sh

`./setup/setup_spdk_with_latency_test.sh <skip_verify>`

+ `skip_verify`

值为 0 或 1。

是否跳过第一次编译和单元测试，在没有开启 LATENCY_LOG 宏的情况下。开启可以验证克隆的 SPDK 版本是否正确、能否正常运行。

#### configure_host.sh

`./setup/configure_host.sh`

配置 Host 端。

#### configure_target.sh

`./setup/configure_target.sh <is_100g>`

+ `is_100g`

值为 0 或 1。

网速是否为 100 Gbps，是为 1，mtu = 9000；否为 0，mtu = 1500；通过 mtu 的大小来获取目标网口的 IP。

#### run_perf_test.sh

`./setup/run_perf_test.sh <cloudlab_username> <hostname> <run_app> <node_num> <file_path> <io_queue_depth> <io_size> <workload> <run_time>`

+ `cloudlab_username`

CS0522。

+ `hostname`

Host 的主机名。值如 `amd263.utah.cloudlab.us`。

+ `run_app`

值为 `perf`或 `perf_rep`。

+ `node_num`

所有节点个数。

+ `file_path`

保存 Target 节点的 local IP 文件，即建立 RDMA 连接的网口。

+ `io_queue_depth`

perf 参数，值可为 `256`。

+ `io_size`

perf 参数，值可为 `4096`。

+ `workload`

perf 参数，值可为 `randread`、`randwrite`、`randrw`，`randrw` 会自动添加 `-M 50`。

+ `run_time`

perf 参数，值的单位为`秒`。

#### get_run_output.sh

`./setup/get_run_output.sh <cloudlab_username> <node_num> [hostnames...]`

+ `cloudlab_username`

CS0522。

+ `node_num`

所有节点个数。

+ `hostnames...`

所有节点主机名。

### 2.3. 全自动配置运行

**本地机器**运行 `./setup/setup_all_nodes.sh`。

```bash
# example
./setup/setup_all_nodes.sh 1 1 CS0522 2 amd263.utah.cloudlab.us amd250.utah.cloudlab.us
```

运行 perf 和 perf_rep，通过本地机器运行 `run_perf_test.sh` 并通过 `get_run_output.sh` 获取结果。

```bash
# example
./setup/run_perf_test.sh CS0522 amd263.utah.cloudlab.us perf 2 ./setup/setup_output/nodes_local_ip.txt 256 4096 randwrite 5
```

### 2.4. 半自动配置运行

**远程机器**分别运行 `./setup/*.sh`。

1. 远程机器 clone SPDK v24.05.x 至目录 `/opt/Workspace` 下；
2. 按照 SPDK 说明更新子模块；
3. 删除原有 `.git` 仓库，新建仓库，并将仓库与 `nof-rep` 仓库关联；
4. `git fetch` 并切换 `git checkout` 至 `latency_test` 分支；
5. 运行 `./setup/setup_rdma.sh 1 1`；
6. 运行 `./setup/setup_spdk_with_latency_test.sh 1`；
7. Host 端运行 `./setup/configure_host.sh`；
8. Target 端运行 `./setup/configure_target.sh`；
9. 运行 perf 和 perf_rep。

