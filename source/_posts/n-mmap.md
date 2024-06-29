---
title: 【学习笔记】mmap 内存映射
tags:
  - mmap
  - Linux
  - C++
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - C++
comments: false
cover: false
date: 2024-06-29 19:44:34
---

mmap 可以把磁盘文件的一部分直接映射到内存，这样文件中的位置直接就有对应的内存地址，对文件的读写可以直接用指针而不需要 read/write 函数。

<!-- more -->

## mmap 简介

mmap 可以把磁盘文件的一部分直接映射到内存，这样文件中的位置直接就有对应的内存地址，对文件的读写可以直接用指针而不需要 read/write 函数。


## mmap 函数介绍

```cpp
void *mmap(void *addr, size_t len, int prot,
		   int flags, int filedes, __off_t offset);

int munmap(void *addr, size_t len);
```

### 参数

* `addr`: 内存上的映射地址，给内核一个提示（建议），从（内存上）什么地址开始映射。建立映射后，真正的映射首地址通过返回值得到。如果 `addr` 为空，则内核自己选择合适的地址。

* `len`: 需要映射的那部分文件的长度（多少个字节），在内存中需要相同的大小。

* `prot`: 4 个取值
  * `PROT_EXEC`: 映射部分可执行，如动态库
  * `PROT_READ`: 映射部分可读
  * `PROT_WRITE`: 映射部分可写
  * `PROT_NONE`: 映射部分不可访问

* `flags`: 2 个取值
  * `MAP_SHARED`: 多个进程对相同映射文件共享
  * `MAP_PRIVATE`: 多个进程对相同映射文件不共享

* `filedes`: 文件描述符。文件描述符是内核为了高效管理已经被打开的文件的索引，是一个非负整数，用于指代被打开的文件。文件描述符可以通过系统函数 `open()` 或 `create()` 获取，也可以从父进程继承，用于进行文件操作或网络通信。

* `offset`: 从文件的什么位置开始映射，必须是页（内存上）大小的整数倍。

### 返回值

返回值为 `void*` 指针，因此可以进行各种类型转换，如 `int*`、`char*` 等。C++ 中不能直接 `int* p = mmap()`，需要对 `mmap()` 进行类型转换，如 `int* p = (int*)mmap()`。

### 需要的头文件

```cpp
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
```



---

## 实例

### 1. 简单对文件进行修改

```cpp
#include <iostream>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <cstring>

int main()
{
    std::string filename = "./dataset/test.txt";
    int len = 12;
    // 获取文件描述符
    int fd = open(filename.c_str(), O_RDWR);
    // 内存映射
    auto* p = (char*)mmap(NULL, len, PROT_WRITE | PROT_READ, MAP_SHARED, fd, 0);
    // 直接对内存进行修改
    for (int i = 0; i < len; i++)
    {
        p[i] = 'a' + i;
    }
    // 关闭内存映射
    munmap(p, len);
    return 0;
}
```

### 2. 读取 bvecs 文件

```cpp
std::vector<std::vector<float>> mmap_read_bvecs(std::string &filename)
{
    // num
    int num = 100;
    // length
    int dim = 128;
    // 多少个字节
    int len = (dim + 4) * num;
    // 获取文件描述符
    int fd = open(filename.c_str(), O_RDWR);
    // 打开失败
    if (fd == -1)
    {
        exit(EXIT_FAILURE);
    }
    // 内存映射，获得指针
    u_char* p = (u_char*)mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

    // 存储数据
    std::vector<std::vector<float>> data;
    // 获取数据
    for (int i = 0; i < num; i++)
    {
        std::vector<float> vec(dim);
        for (int j = 0; j < dim; j++)
        {
            vec[j] = (float)(p[i * (dim + 4) + 4 + j]);
        }
        data.push_back(std::move(vec));
    }

    // 关闭映射
    munmap(p, len);

    return data;
}
```