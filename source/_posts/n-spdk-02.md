---
title: 【学习笔记-存储】spdk（二）：spdk 安装与部署
tags:
  - 存储
  - spdk
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2024-08-29 15:46:24
---

记录 spdk 的安装和部署过程。

<!-- more -->

## 文件准备 

需要下载的文件：

* [spdk-24.05.x.zip](https://codeload.github.com/spdk/spdk/zip/refs/heads/v24.05.x)
* [isa-l_crypto-08297dc3e76d65e1bad83a9c9f9e49059cf806b5.zip](https://codeload.github.com/intel/isa-l_crypto/zip/08297dc3e76d65e1bad83a9c9f9e49059cf806b5)
* [libvfio-user-a646db0b7d65c774a0b9415ea582735e7c2d2eb6.zip](https://codeload.github.com/nutanix/libvfio-user/zip/a646db0b7d65c774a0b9415ea582735e7c2d2eb6)
* [dpdk-08f3a46de70afff49f55d175de690b5ad7e4a44d.zip](https://codeload.github.com/spdk/dpdk/zip/08f3a46de70afff49f55d175de690b5ad7e4a44d)
* [xnvme-3834fd860d40b6a3608aae11f9ceb017a0c93b29.zip](https://codeload.github.com/xnvme/xnvme/zip/3834fd860d40b6a3608aae11f9ceb017a0c93b29)
* [ocf-d1d6d7cb5f55b616d2aa5123f84ce4ece10fdb0b.zip](https://codeload.github.com/Open-CAS/ocf/zip/d1d6d7cb5f55b616d2aa5123f84ce4ece10fdb0b)
* [isa-l-6f420b14a1e3e091bc9d15f508a54f82c007483c.zip](https://codeload.github.com/spdk/isa-l/zip/6f420b14a1e3e091bc9d15f508a54f82c007483c)
* [intel-ipsec-mb-935a3802883249ba3b12e566833994af7991e808.zip](https://codeload.github.com/spdk/intel-ipsec-mb/zip/935a3802883249ba3b12e566833994af7991e808)


## 安装与构建

### clone 项目源码

```bash
# pwd: ~/Workspace/spdk-24.05.x

# clone 源码
git clone -b v24.05.x git@github.com:spdk/spdk.git
# 或者直接下载 zip 文件（推荐）
# spdk-24.05.x.zip
```

### 更新子模块

```bash
# 更新子模块
git submodule update --init
# 建议手动 clone 子模块（网络原因）
# 1. 根据 Github repo 中 spdk 版本对应的 submodule 版本，下载对应 zip 文件
# spdk-v24.05.x 所需版本：
# dpdk-08f3a46de70afff49f55d175de690b5ad7e4a44d.zip
# intel-ipsec-mb-935a3802883249ba3b12e566833994af7991e808.zip
# isa-l-6f420b14a1e3e091bc9d15f508a54f82c007483c.zip
# isa-l_crypto-08297dc3e76d65e1bad83a9c9f9e49059cf806b5.zip
# libvfio-user-a646db0b7d65c774a0b9415ea582735e7c2d2eb6.zip
# ocf-d1d6d7cb5f55b616d2aa5123f84ce4ece10fdb0b.zip
# xnvme-3834fd860d40b6a3608aae11f9ceb017a0c93b29.zip
# 2. 解压后删掉 '-xxx'，复制到 spdk 根目录下
```

### 安装依赖

```bash
# 安装各种依赖包
sudo ./scripts/pkgdep.sh -r
# 出于网络原因，可以修改 sh 脚本，添加临时源
# scripts/pkgdep/ubuntu.sh
pip3 install ninja -i https://pypi.doubanio.com/simple
pip3 install meson -i https://pypi.doubanio.com/simple
pip3 install pyelftools -i https://pypi.doubanio.com/simple
pip3 install ijson -i https://pypi.doubanio.com/simple
pip3 install python-magic -i https://pypi.doubanio.com/simple
pip3 install grpcio -i https://pypi.doubanio.com/simple
pip3 install grpcio-tools -i https://pypi.doubanio.com/simple
pip3 install pyyaml -i https://pypi.doubanio.com/simple
pip3 install Jinja2 -i https://pypi.doubanio.com/simple
pip3 install tabulate -i https://pypi.doubanio.com/simple
```

### 打上 patch、修改代码

位于 `lib/log/log.c`。添加和修改以下代码：

```c
// lib/log.log.c

void spdk_vlog(enum spdk_log_level level, const char *file, const int line, const char *func,
	  const char *format, va_list ap)
{
	int severity = LOG_INFO;
	char *buf, _buf[MAX_TMPBUF], *ext_buf = NULL;
	char timestamp[64];
	va_list ap_copy;    // 添加代码
	int rc;

	if (g_log) {
		g_log(level, file, line, func, format, ap);
		return;
	}

	if (level > g_spdk_log_print_level && level > g_spdk_log_level) {
		return;
	}

	severity = spdk_log_to_syslog_level(level);
	if (severity < 0) {
		return;
	}

	buf = _buf;

	va_copy(ap_copy, ap);   // 添加代码
	rc = vsnprintf(_buf, sizeof(_buf), format, ap);
	if (rc > MAX_TMPBUF) {
		/* The output including the terminating was more than MAX_TMPBUF bytes.
		 * Try allocating memory large enough to hold the output.
		 */
		rc = vasprintf(&ext_buf, format, ap_copy);      // 修改代码：ap -> ap_copy
		if (rc < 0) {
			/* Failed to allocate memory. Allow output to be truncated. */
		} else {
			buf = ext_buf;
		}
	}

	if (level <= g_spdk_log_print_level) {
		get_timestamp_prefix(timestamp, sizeof(timestamp));
		if (file) {
    // ......
```

### 构建项目

```bash
# 构建
./configure --with-rdma
make
```

### 执行单元测试

```bash
./test/unit/unittest.sh

# 正确结果：
# =====================
# All unit tests passed
# =====================
# WARN: lcov not installed or SPDK built without coverage!
# WARN: neither valgrind nor ASAN is enabled!
```