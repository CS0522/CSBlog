---
title: 【学习笔记】gRPC（一）：编译与安装
tags:
  - 存储
  - RPC
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
comments: false
cover: false
date: 2025-01-21 01:45:05
---

gRPC 编译与安装。

<!-- more -->

## 系统环境

* Ubuntu 20.04
* gRPC v1.34.0

gRPC 该版本在 Ubuntu 22.04 下编译出错。

---

## 安装依赖、Clone 仓库

```bash
sudo apt install -y build-essential autoconf libtool \
                pkg-config cmake make git vim

git clone --recurse-submodules -b v1.34.0 --depth 1 \
          --shallow-submodules https://github.com/grpc/grpc.git

# 获取已经打包好的
wget https://github.com/CS0522/CSBlog/releases/download/coroutine_kvdb_files/grpc-1.34.0.zip
unzip grpc-1.34.0.zip
```

--- 

## 编译

一起编译其他依赖库，不需要额外安装。

```bash
cd grpc

# 建议不安装在 /usr/local
export MY_INSTALL_DIR=$HOME/.local
mkdir -p $MY_INSTALL_DIR
export PATH="$MY_INSTALL_DIR/bin:$PATH"

mkdir -p cmake/build
pushd cmake/build
cmake -DgRPC_INSTALL=ON \
      -DgRPC_BUILD_TESTS=OFF \
      -DCMAKE_INSTALL_PREFIX=$MY_INSTALL_DIR \
      ../..
make -j16
sudo make install
popd

# 再将 PATH 写入 /etc/profile 或 ~/.bashrc
vim ~/.bashrc
export export PATH="$HOME/.local/bin:$PATH"
source ~/.bashrc
```

---

## proto 文件生成命令

```bash
protoc --cpp_out=./include --grpc_out=./include --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` -I./proto db.proto
```