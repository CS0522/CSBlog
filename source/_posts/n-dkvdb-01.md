---
title: 【学习笔记】分布式 KVDB（一）：bRPC 框架编译安装
tags:
  - 存储
  - 分布式 KVDB
  - bPRC
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2024-12-03 15:12:28
---

bRPC 编译、安装与样例测试。

<!-- more -->

## 系统环境

+ Ubuntu 18.04 以上
+ gcc 4.x ~ 11.x
+ brpc-1.5.0
+ 3 个特定版本的关键依赖：

    - gflags-2.2.2
    - protobuf-3.20.0
    - glog-0.4.0
+ 后文的依赖和项目的路径：

    - `/users/CS0522/bthread-dep/gflags-2.2.2/`
    - `/users/CS0522/bthread-dep/protobuf/`
    - `/users/CS0522/bthread-dep/glog-0.4.0/`
    - `/users/CS0522/custom-brpc/brpc/`

## 安装依赖

### 安装其他依赖

openssl、leveldb、ibverbs，以及**静态链接 leveldb 的依赖**：

```bash
sudo apt install -y libssl-dev libleveldb-dev libibverbs-dev librdmacm-dev libsnappy-dev
```

### 安装关键依赖
**按顺序安装！**

安装 gflags：

```bash
# /users/CS0522/bthread-dep/gflags-2.2.2
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=ON -DGFLAGS_NAMESPACE=google -DCMAKE_HAVE_LIBC_PTHREAD=ON -G "Unix Makefiles" -DBUILD_SHARED_LIBS=1 -DBUILD_STATIC_LIBS=1 ../
make -j16
sudo make install
sudo systemctl daemon-reload
sudo ldconfig
```

安装 protobuf：

```bash
# /users/CS0522/bthread-dep/protobuf/
make clean && make distclean
./configure --prefix=/usr/local
make -j16
sudo make install
sudo systemctl daemon-reload
sudo ldconfig
```

安装 glog：

```bash
# /users/CS0522/bthread-dep/glog-0.4.0/
./autogen.sh
./configure CXXFLAGS="-std=c++11"
make -j16
sudo make install
```

## 编译 brpc

```bash
# /users/CS0522/custom-brpc/brpc/
sudo chmod 777 ./tools/*.sh
mkdir build && cd build
cmake ..
make -j16
sudo make install
```

## echo 样例测试

### 修改 CMakeLists.txt

新的压缩包 `custom-bprc.zip` 中已经修改好：

```bash
# /users/CS0522/custom-brpc/brpc/example/echo_c++/
vim CMakeLists.txt

# 添加 glog
find_library(GLOG_LIB NAMES glog/logging.h)
find_path(GLOG_INCLUDE_PATH NAMES glog)
if (NOT GLOG_LIB)
    set(GLOG_LIB "")
    set(GLOG_INCLUDE_PATH "")
endif()
include_directories(${GLOG_INCLUDE_PATH})

# set DYNAMIC_LIB 中添加 GLOG_LIB
set(DYNAMIC_LIB
    ...
    ${GLOG_LIB}
    dl
    )

# 保存退出
```

### 编译

```bash
# /users/CS0522/custom-brpc/brpc/example/echo_c++/
mkdir build && cd build
cmake ..
make  -j16
```

### 运行

```bash
# /users/CS0522/custom-brpc/brpc/example/echo_c++/build/
./echo_server &
./echo_client
```