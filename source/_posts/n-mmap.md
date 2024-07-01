---
title: 【学习笔记】虚拟内存 + mmap 内存映射
tags:
  - mmap
  - 虚拟内存
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

---

## 1. 虚拟内存

[https://welkinx.com/2020/02/07/mmap/](https://welkinx.com/2020/02/07/mmap/)

### 1.1 Linux 增加交换空间

未开启交换空间：

```bash
sudo su
# 查看内存使用情况
free -h

# 创建 swap 文件
# 单位 bs 为 G，大小 count 为 20，创建 20G 交换空间
dd if=/dev/zero of=/swapfile bs=1G count=20

# 激活 swap 文件
chmod 600 swapfile
mkswap swapfile

# 开启 swap
swapon swapfile

free -h
# 查看已开启的交换空间
swapon --show
```

已开启交换空间，重新修改 swap 大小：

```bash
# 查看内存使用情况
free -h

# 关闭指定 swap
swapoff /swapfile

# 重新分配
fallocate -l 30G /swapfile

chmod 600 swapfile
mkswap swapfile
swapon swapfile
```

### 1.2 创建虚拟内存实例

#### 1.2.1 创建虚拟内存、在虚拟内存中读入大文件数据并存储

注意 placement new 表达式的用法，其中 new(place) 中 place 为指针，指向预分配的内存地址。

<details>
  <summary>点击查看</summary>

  ```cpp
  #include <iostream>
  #include <fstream>
  #include <sys/mman.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <cstring>
  #include <vector>
  
  /**
   * TODO
   * 用 mmap 实现虚拟大内存，并构建一个接口调用；
   * 在这块虚拟大内存中读入数据集并保存，用于建图。
  */
  
  
  /**
   * @name: create_virtual_mem
   * @msg: 创建虚拟内存，并返回指向该内存地址的指针
   * @param {off_t&} mem_size: 虚拟内存大小
   * @param {int&} fd: 文件描述符，main 函数中指定
   * @return {void*}: 虚拟内存指针
   */
  void* create_virtual_mem(off_t &mem_size, int &fd)
  {
      // 使用 /dev/zero 作为匿名内存映射的数据源
      fd = open("/dev/zero", O_RDWR);
      void* v_mem = mmap(nullptr, mem_size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, fd, 0);
      // 映射失败
      if (v_mem == MAP_FAILED) 
      {
          std::cout << "Create virtual memory failed. Reason: " << strerror(errno) << std::endl;
          exit(EXIT_FAILURE);
      }
  
      // 文件和映射统一关闭
  
      // 返回虚拟内存指针
      return v_mem;
  }
  
  
  /**
   * @name: release_mem
   * @msg: 释放虚拟内存以及内存映射
   * @param {void*} v_mem: 指向虚拟内存的指针
   * @param {off_t&} mem_size: 虚拟内存大小
   * @param {int&} fd: 文件描述符，用于关闭虚拟内存打开的 /dev/zero
   * @param {u_char* &} file_map: 读取大文件时的内存映射的指针
   * @param {off_t &} file_size: 被映射的大文件的长度（字节）
   * @return {void}
   */
  void release_mem(void* v_mem, off_t &mem_size, int &fd, u_char* &file_map, off_t &file_size)
  {
      munmap(v_mem, mem_size);
      close(fd);
      // 关闭 read data 映射
      munmap(file_map, file_size);
      // 关闭文件
      close(fd);
  }
  
  
  /**
   * @name: mmap_read_bvecs
   * @msg: 通过 mmap 内存映射读取 *.bvecs 文件
   * @param {string} &filename: 文件名
   * @param {void*} &v_mem: 虚拟内存指针
   * @param {u_char* &} file_map: 读取大文件时的内存映射的指针
   * @param {off_t &} file_size: 被映射的大文件的长度（字节）
   * @param {std::vector<std::vector<float>>* &} data: 存储数据
   * @return {void}
   */
  void mmap_read_bvecs(std::string &filename, void* v_mem, u_char* &file_map, off_t &file_size,
                                                   std::vector<std::vector<float>>* &data)
  {
      // 获取文件描述符
      int fd = open(filename.c_str(), O_RDWR);
      // 打开失败
      if (fd == -1)
      {
          perror(("Open \"" + filename + "\" failed. Reason").c_str());
          exit(EXIT_FAILURE);
      }
      // 获取文件信息
      struct stat fs;
      // 获取失败
      if (fstat(fd, &fs) == -1) {
          perror("Get file information filed. Reason");
          close(fd);
          exit(EXIT_FAILURE);
      }
  
      // 文件大小
      file_size = fs.st_size;
      std::cout << "File size: " << file_size << std::endl;
      
      // read data 映射，获得指针
      file_map = (u_char*)mmap(nullptr, file_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
      // 映射失败
      if (file_map == MAP_FAILED) 
      {
          perror("Memory mapping failed. Reason");
          close(fd);
          exit(EXIT_FAILURE);
      }
  
      // 访问数据
      long long byte_count = 0LL;
      // vector number
      long long num = 0;
      std::cout << "Reading..." << std::endl;
      while(byte_count < file_size)
      {
          // vector dimension
          int dim;
          u_char* q = file_map + byte_count;
          dim = *(int*)q;
          q += sizeof(int);
          // 物理内存内保存单个数据
          std::vector<float> vec(dim);
          for (int i = 0; i < dim; i++)
          {
              vec[i] = (float)*(q + i);
          }
          // 存储到 data 中
          data->push_back(std::move(vec));
   
          byte_count += sizeof(int) + dim * sizeof(u_char);
          num += 1;
      }
  
      // 文件和映射需要统一关闭
  }
  
  
  int main()
  {
      std::string filename = "./dataset/bigann_learn.bvecs";
      
      // 创建虚拟内存
      int fd_v_mem;
      off_t mem_size = 1024 * 1024 * 1024 * 20LL;
      auto v_mem = create_virtual_mem(mem_size, fd_v_mem);
      
      // 读入数据
      u_char *file_map = nullptr;
      off_t file_size = 0LL;
      // 在虚拟内存中开辟空间存储
      // 注意 placement new 表达式的用法
      std::vector<std::vector<float>>* data = new(v_mem) std::vector<std::vector<float>>();
      mmap_read_bvecs(filename, v_mem, file_map, file_size, data);
      
      // output
      auto vec_num = data->size();
      auto vec_dim = (*data)[0].size();
      std::cout << "vector number: " << vec_num << std::endl;
      std::cout << "vector dimension: " << vec_dim << std::endl;
      // for (size_t i = 0; i < vec_num; ++i)
      // {
      //     std::cout << "[";
      //     for (size_t j = 0; j < vec_dim; ++j)
      //     {
      //         std::cout << (*data)[i][j] << ((j == vec_dim - 1) ? "" : ", ");
      //     }
      //     std::cout << "]\n";
      // }
  
      // 关闭映射和虚拟内存
      release_mem(v_mem, mem_size, fd_v_mem, file_map, file_size);
  
      return 0;
  }
  ```

</details>

---


## 2. mmap

[https://www.cnblogs.com/huxiao-tee/p/4660352.html](https://www.cnblogs.com/huxiao-tee/p/4660352.html)

### 2.1 mmap 简介

mmap 可以把磁盘文件的一部分直接映射到内存，这样文件中的位置直接就有对应的内存地址，对文件的读写可以直接用指针而不需要 read/write 函数。

此函数的作用是创建一个新的**虚拟内存**区域，并将指定的对象映射到此区域。

内存映射（Memory Mapping）将文件内容映射到进程的虚拟地址空间。在这种机制下，文件可以被视为内存的一部分，从而允许程序直接对这部分内存进行读写操作，而无需传统的文件 I/O 调用，**从而减少系统的用户态、内核态切换**，减少切换开销。通过这种方式，文件内容可以通过指针直接访问，就像访问普通的内存数组一样，这极大地提高了文件操作的效率和直观性。

映射时，操作系统将文件的一部分或全部内容映射到虚拟内存地址空间。这些虚拟地址与物理内存地址相关联，但并不是所有数据立即加载到物理内存中。

当访问映射的地址时，如果对应数据不在物理内存中，操作系统会自动从磁盘加载所需的数据页到物理内存中（这称为“页错误”处理）。

### 2.2 mmap 特点
#### 2.2.1 数据持久化

通过 mmap 映射的数据通常来自文件系统中的文件。这意味着数据是持久化的，即使程序终止，文件中的数据依然存在。当你通过映射的内存区域修改数据时，这些更改最终会反映到磁盘上的文件中。

#### 2.2.2 大文件读写

mmap 特别适合于需要频繁读写大文件的场景，因为它可以减少磁盘 I/O 操作的次数。它也允许文件的一部分被映射到内存中，这对于处理大型文件尤为有用。

#### 2.2.3 性能和效率

映射文件到内存可以提高文件访问的效率，尤其是对于随机访问或频繁读写的场景。系统可以利用虚拟内存管理和页缓存机制来优化访问。

#### 2.2.4 同步和一致性

使用 mmap 时，必须考虑到文件内容的同步问题。例如，使用 `msync` 调用来确保内存中的更改被同步到磁盘文件中。

#### 2.2.5 页缓存

使用 mmap 映射文件到内存时，操作系统利用**页缓存**（提升文件读写效率。在内存上，与文件中的数据块进行绑定。文件被划分为多个页大小的数据块）来优化对这些文件数据的访问。页缓存是操作系统的一部分，用于存储从磁盘读取的数据页。
访问 mmap 映射的文件时，并不是每次读取都会直接触及磁盘。如果所需数据已经在页缓存中（由于之前的读取操作），则直接从缓存中获取数据，而不需要磁盘 I/O。


### 2.3 mmap 函数介绍

```cpp
void *mmap(void *addr, size_t len, int prot,
		   int flags, int filedes, __off_t offset);

int munmap(void *addr, size_t len);
```

#### 2.3.1 参数

* `addr`: 内存上的映射地址，给内核一个提示（建议），从（内存上）什么地址开始映射。建立映射后，真正的映射首地址通过返回值得到。如果 `addr` 为空，则内核自己选择合适的地址。

* `len`: 需要映射的那部分文件的长度（多少个字节），代表将文件中多大的部分映射到内存。

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

#### 2.3.2 返回值

返回值为 `void*` 指针，因此可以进行各种类型转换，如 `int*`、`char*` 等。C++ 中不能直接 `int* p = mmap()`，需要对 `mmap()` 进行类型转换，如 `int* p = (int*)mmap()`。

#### 2.3.3 需要的头文件

```cpp
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
```

---

### 2.4 mmap 实例

#### 2.4.1 简单对文件进行修改

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

#### 2.4.2 读取 bvecs 文件

```cpp
std::vector<std::vector<float>> mmap_read_bvecs(std::string &filename)
{
    // 获取文件描述符
    int fd = open(filename.c_str(), O_RDWR);
    // 打开失败
    if (fd == -1)
    {
        perror(("Open \"" + filename + "\" failed. Reason").c_str());
        exit(EXIT_FAILURE);
    }
    // 获取文件信息
    struct stat fs;
    // 获取失败
    if (fstat(fd, &fs) == -1) {
        perror("Get file information filed. Reason");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // 文件大小
    off_t file_size = fs.st_size;
    std::cout << "File size: " << file_size << std::endl;
    
    // 内存映射，获得指针
    u_char* p = (u_char*)mmap(NULL, file_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    // 映射失败
    if (p == MAP_FAILED) 
    {
        perror("Memory mapping failed. Reason");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // 访问数据
    long long byte_count = 0LL;
    // 获取 vector number
    int num = 0;
    while(byte_count < file_size)
    {
        // vector dimension
        int dim;
        u_char *q = p + byte_count;
        dim = *(int*)q;
        q += sizeof(int);
        // output vector
        // std::cout << "[";
        for (int i = 0; i < dim; i++)
        {
            std::cout << (float)*(q + i) << ((i == dim - 1) ? "]\n" : ", ");
        }

        byte_count += sizeof(int) + dim * sizeof(u_char);
        num += 1;
    }

    // output vector number
    std::cout << "vector num: " << num << std::endl;
    // 关闭映射
    munmap(p, file_size);
    // 关闭文件
    close(fd);

    // return data;
    return {};
}
```