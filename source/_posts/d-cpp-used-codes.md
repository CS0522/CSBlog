---
title: 【文档小记】记录 C++ 中使用过的有用代码
tags:
  - C++
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2024-06-29 21:46:07
---

记录 C++ 中使用过的有用代码，以后应该还用的上。

<!-- more -->

## const char*, string 与 char* 的转化

```cpp
// string 与 const char* 转换
string s = "abcd";
const char *c_s = s.c_str();

// const char* 转换 string
const char* p = "abcd";
string s(p);

// string 与 char* 的转换
string s = "abcd";
char* c;
const int len = s.length();
c = new char[len + 1];
strcpy(c, s.c_str());
```

## 减少 vector 增加元素的开销

```cpp
std::vector<std::pair<int, int>> vec;

// 1. emplace_back
vec.emplace_back(1, 2);

// 2. move
auto p = std::make_pair(1, 2);
vec.push_back(std::move(p));
```

## 获取文件描述符

```cpp
#include <fcntl.h>

int fd = open(filename.c_str(), O_RDWR);
// 打开失败的返回值为 -1
```


## 读取 ivecs 数据文件

ivecs 格式每个向量构成：`4 + dim * 4` 字节，`4` 为一个 `int`。

```cpp
std::vector<std::vector<int>> read_ivecs(const std::string &filename)
{
    std::ifstream input(filename, std::ios::binary);
    if (!input.is_open())
    {
        std::cerr << "无法打开文件: " << filename << std::endl;
        exit(EXIT_FAILURE);
    }

    std::vector<std::vector<int>> data;
    while (!input.eof())
    {
        int dim;
        input.read(reinterpret_cast<char *>(&dim), sizeof(int));
        std::vector<int> vec(dim);
        input.read(reinterpret_cast<char *>(vec.data()), dim * sizeof(int));
        if (input)
        {
            data.push_back(std::move(vec));
        }
    }
    return data;
}
```


## 读取 bvecs 数据文件

bvecs 格式每个向量构成：`4 + dim * 1` 字节，`4` 为一个 `int`。

```cpp
std::vector<std::vector<float>> read_bvecs(const std::string &filename)
{
    std::ifstream input(filename, std::ios::binary);
    if (!input.is_open())
    {
        std::cerr << "Open " << filename << "failed. " << std::endl;
        exit(EXIT_FAILURE);
    }

    std::vector<std::vector<float>> data;
    while (!input.eof())
    {
        int dim;
        input.read((char *)&dim, sizeof(int));
        
        std::vector<float> vec(dim);
        std::vector<u_char> buff(dim);
        input.read((char *)buff.data(), dim * sizeof(u_char));

        // u_char to float
        for (int i = 0; i < dim; i++)
        {
            vec[i] = (float)(buff[i]);
        }

        data.push_back(std::move(vec));
    }
    return data;
}
```

mmap 内存映射读取部分 bvcecs 向量

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