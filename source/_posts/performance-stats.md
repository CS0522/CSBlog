---
title: 【学习笔记】Linux C++ 性能分析
tags:
  - C++
  - valgrind
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - valgrind
comments: false
cover: false
date: 2024-05-09 17:15:54
---

C++ 项目需要统计性能指标，学习并使用性能分析工具。

<!-- more -->

---

## Valgrind

Valgrind 是一种开源工具套件，旨在帮助开发人员调试、分析和优化程序的内存使用和执行性能。Valgrind 包括多个工具：Memcheck、Cachegrind、Callgrind、Helgrind、DRD、Massif 等。这些工具可用于检测内存泄漏、越界读写、未初始化变量、线程竞争、函数调用关系、缓存使用情况、内存使用情况等问题。这些问题通常是程序崩溃或性能下降的主要原因。Valgrind 的主要优点是它可以帮助开发人员快速找到并解决程序中的内存和性能问题。它是开发和测试过程中的有用工具，特别是对于大型和复杂的代码库。缺点是 Valgrind 会降低程序的性能，因此不建议在生产环境中使用它。

Valgrind 工具集包含以下几个主要工具：

* Memcheck: 用于检测内存泄漏、内存错误等问题。
* Cachegrind: 用于检测程序中的缓存命中率、分支预测错误等问题。
* Callgrind: 用于生成函数调用图和性能分析报告。
* Helgrind: 用于检测多线程程序中的竞争条件、死锁等问题。
* DRD: 用于检测多线程程序中的并发错误。
* Massif: 用于检测程序中的内存使用情况。

### Valgrind 命令

> [Valgrind Docs](https://valgrind.org/docs/manual/manual-core.html#manual-core.options)

```bash
valgrind [valgrind-options] your-prog [your-prog-options]
```

valgrind-options:
* `--tool=`: choose which tool to use, like `--tool=challvalgrind`
* `--trace-children=<yes|no>`: 跟踪到子进程里去，默认请况不跟踪
* `--log-file=filename`: 将输出的信息写入到 `filename.PID` 的文件里
* `--log-file-exactly=filename`: 指定就输出到 `filename` 文件
* `--separate-threads=yes`: 多线程

### Callgrind 使用

#### cpp 源文件
  
```cpp
// test_wadg_search.cpp
  ```

#### 编译

编译时加上 `-g` 参数，方便显示行数，如：
```bash
g++ -O0 -g -std=c++11 -lpthread test_wadg_search.cpp -o test_wadg_search
```
本项目中是 CMake，如：
```bash
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g")
```

#### 运行 callgrind

```bash
valgrind --tool=callgrind --separate-threads=yes ./build/tests/test_wadg_search ./dataset/sift/sift_base.fvecs ./dataset/sift/sift_query.fvecs ./nsg_graph/sift.nsg 200 200 ./search_result/sift_200nn.ivecs
```

得到 `callgrind.out.PID` 文件。


#### callgrind_annotate 分析文件

使用 `callgrind_annotate` 获取 `callgrind` 生成的输出文件，并以易于阅读的形式打印信息。

```bash
callgrind_annotate [options] <callgrind.out.PID>
```

`options` 可以包括：
  
* `--tree=both`: 显示调用树的入口和出口视图。
* `--inclusive=yes`: 显示包括调用子函数在内的总时间。
* `--threshold=<percentage>`: 只显示占总时间超过某个百分比的函数。

想要一个包含子函数时间的更详细的报告，可以运行：

```bash
callgrind_annotate --tree=both --inclusive=yes <callgrind.out.PID>
```


#### kcachegrind 分析文件

本人 Ubuntu 22.04 电脑上运行存在问题，“打开”不显示文件。


#### 使用 gprof2dot 可视化

* 安装 `gprof2dot` & `graphviz`：
  ```bash
  conda create -n "gprof2dot-env" python==3.7
  conda activate "gprof2dot-env"
  pip install gprof2dot
  pip install graphviz
  ```

* 转化为 `png` 图片：
  ```bash
  # 一定注意是原始的 callgrind.out.* 文件
  # 转化为 dot 脚本
  gprof2dot -f callgrind callgrind.out.PID > callgrind.dot

  # 输出为 png
  dot -Tpng callgrind.dot -o callgrind.png

  # 可以简写
  prof2dot -f callgrind callgrind.out.PID | dot -Tpng -o callgrind.png
  ```


---

## gprof

Gprof 是 Linux 下一个强有力的程序分析工具。它能够以“日志”的形式记录程序运行时的统计信息：程序运行中各个函数消耗的时间和函数调用关系，以及每个函数被调用的次数等等。


### gprof 缺陷

* 只能分析应用程序在运行过程中所消耗掉的用户时间，无法得到程序内核空间的运行时间，如 `sleep()`
* gprof 不支持多线程应用，多线程下只能采集主线程性能数据。原因是 gprof 采用 ITIMER_PROF 信号，在多线程内只有主线程才能响应该信号。但是有一个简单的方法可以解决这一问题：http://sam.zoy.org/writings/programming/gprof.html


### gprof 使用

#### cpp 源文件

```cpp
// test_wadg_search.cpp
```

#### 编译

编译时，加上 `-pg` 参数，如：
```bash
g++ -O0 -pg -std=c++11 -lpthread test_wadg_search.cpp -o test_wadg_search
```
本项目中是 CMake，如：
```bash
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -pg")
```

#### 运行程序

```bash
test_wadg_search [args]
```

运行后在当前工作目录下产生 `gmon.out` 文件。

#### 运行 gprof

将生成的 `gmon.out` 文件输出为可读的文本文件。

```bash
gprof ./build/tests/test_wadg_search ./anals/gmon.out > gprof.profile
```

#### gprof 报告说明

| %time | Cumulative seconds | Self seconds | Calls | Self TS/call | Total TS/call | name |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 该函数消耗时间占程序所有时间百分比 | 程序的累积执行时间（只是包括gprof能够监控到的函数） | 该函数本身执行时间（所有被调用次数的合共时间） | 函数被调用次数 | 函数平均执行时间（不包括被调用时间）（函数的单次执行时间） | 函数平均执行时间（包括被调用时间）（函数的单次执行时间） | 函数名 |

Call Graph 字段含义：

| Index | %time | Self | Children | Called | Name |
|:--:|:--:|:--:|:--:|:--:|:--:|
| 索引值 | 函数消耗时间占所有时间百分比 | 函数本身执行时间 | 执行子函数所用时间 | 被调用次数 | 函数名 |


#### 使用 gprof2dot 可视化

* 安装 `gprof2dot` & `graphviz`：
  ```bash
  conda create -n "gprof2dot-env" python==3.7
  conda activate "gprof2dot-env"
  pip install gprof2dot
  pip install graphviz
  ```

* 转化为 `png` 图片：
  ```bash
  # 一定注意是生成后的 utf-8 文本文件，不是原来的 gmon.out 文件
  # 转化为 dot 脚本
  gprof2dot gprof.profile > gprof.dot

  # 输出为 png
  dot -Tpng gprof.dot -o gprof.png

  # 可以简写
  prof2dot gprof.profile | dot -Tpng -o gprof.png
  ```