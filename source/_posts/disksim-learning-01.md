---
title: 【学习笔记】DiskSim 学习（一）：安装
tags:
  - DiskSim
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - DiskSim
comments: false
cover: false
date: 2023-10-23 20:17:11
---

老东西 DiskSim 环境的配置

<!-- more -->

## 环境
### 系统环境
* Ubuntu14.04 x64 以下
* 测试通过：Ubuntu14.04 x64，Ubuntu10.04 x86
* 老版本需要更换官方源
  ```bash
  # Ubuntu10.04 换源
  ```

### gcc 等工具
* <font color="red">gcc 版本要在 gcc-4.8 以下，越老越好</font>
* 直接安装
  ```bash
  sudo apt-get install gcc g++ build-essential make flex-old bison
  ```

## 安装 DiskSim
### 1. 下载源码包
* [DiskSim](http://www.pdl.cmu.edu/DiskSim/)
* [ssd-add-on](http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/)

### 2. 解压
```bash
tar xfz disksim-4.0-with-dixtrac.tar.gz
cd disksim-4.0
unzip ../ssd-add-on.zip
```

### 3. 下载修改补丁、x64 兼容补丁
* [modify-patch](https://github.com/cighao/disksim-4.0-with-ssdmodel-patch)
* [64bit-patch](https://github.com/cighao/disksim-4.0-with-ssdmodel-64bit-patch)

### 4. 打上补丁
```bash
# disksim-4.0/
patch -p1 < ssdmodel/ssd-patch

patch -p1 < modify-patch

patch -p1 < 64bit-patch
```

## 编译与验证
### Make
```bash
# disksim-4.0/
make
```

### 验证
```bash
cd valid
./runvalid

# 验证 ssdmodel
chmod a+x ./ssdmodel/valid/runvalid
cd ./ssdmodel/valid
./runvalid
```