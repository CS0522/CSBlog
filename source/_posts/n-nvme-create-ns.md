---
title: 【学习笔记】nvme-cli 命令：删除 NS 命名空间并创建
tags:
  - 存储
  - Linux
  - NVMe
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2025-05-11 10:22:09
---

记录一次在服务器上捣鼓好久把 NVMe 盘弄没了，最后误打误撞给弄回来的经验。

<!-- more -->

服务器上存在多个 `nvmeXnY` 命令空间，同时还有未显示的空间（多个命名空间容量之和仍小于盘的容量），
打算把他们合并成一个 `nvmeXn1`，并能达到最大容量。可以采取以下方法：

> 以 `nvme0` 为例

1. 查看 NVMe 盘相关参数

```bash
nvme id-ctrl /dev/nvme0
```

主要查看 `cntlid` 和 `tnvmcap` 参数，如：

```bash
...
cntlid: 0x41
...
tnvmcap: 1600321314816
...
```

获取设备块大小，一般为 `512 bytes` 或 `4096 bytes`，可以通过 `fdisk -l` 查看。

2. 计算逻辑块个数

`N = tnvmcap / block_size = 1600321314816 / 512 = 3125627568`

3. 关键一步：分离 NS 和 Controller

```bash
nvme detach-ns /dev/nvme0 -n 1 -c 0x41
```

4. 删除命名空间

```bash
nvme delete-ns /dev/nvme0 -n 1
```

5. 创建命名空间

```bash
nvme create-ns /dev/nvme0 --nsze=3125627568 --ncap=3125627568 --flbas=0
```

6. 连接 Controller

```bash
nvme attach-ns /dev/nvme0 -n 1 -c 0x41
```

大功告成。