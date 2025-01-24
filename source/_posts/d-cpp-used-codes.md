---
title: 【学习笔记】记录 C++ 中使用过的有用代码
tags:
  - C++
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
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

## mmap 内存映射读取大文件的一部分

```cpp
void mmap_read_bvecs(std::string &filename)
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
}

```


## 打印错误信息

* `void perror(const char *s)` 用来将上一个函数发生错误的原因输出到标准设备 (stderr)。参数 s 所指的字符串会先打印，后面再加上错误原因字符串。此错误原因依照全局变量 error 的值来决定要输出的字符串。在库函数中有个error变量，每个error值对应着以字符串表示的错误类型。当调用函数出错时，该函数已经重新设置了error的值。perror 将输入的一些信息和现在的error所对应的错误一起输出。

* `strerror()` 通过标准错误的标号，获得错误的描述字符串，将单纯的错误标号转为字符串描述，方便用户查找错误。

```cpp
void perror(const char *s);
char* strerror(errno);

// perror
// 获取文件描述符
int fd = open(filename.c_str(), O_RDWR);
// 打开失败
if (fd == -1)
{
    perror(("Open \"" + filename + "\" failed. Reason").c_str());
    exit(EXIT_FAILURE);
}

// strerror
printf("strerror: %s\n", strerror(errno));
```


## 创建虚拟内存

```cpp
int main() {
    off_t size = 1024 * 1024;  // 虚拟内存大小为 1MB
    int fd = open("/dev/zero", O_RDWR);  // 使用 /dev/zero 作为匿名内存映射的数据源
    void* ptr = mmap(nullptr, size, PROT_READ | PROT_WRITE, MAP_PRIVATE, fd, 0);

    if (ptr == MAP_FAILED) {
        std::cerr << "mmap failed" << std::endl;
        return 1;
    }

    // 可以使用 ptr 来访问这块虚拟内存
    // ...

    munmap(ptr, size);  // 当不再需要这块虚拟内存时，记得释放它
    close(fd);  // 关闭文件描述符
}
```

## 虚拟内存中创建 vector 数组

在虚拟内存中使用**定位 new 运算符（placement new）**，最后手动在虚拟内存中调用 std::vector 的析构函数

```cpp
// 使用虚拟内存创建 std::vector
std::vector<int>* vec = new (ptr) std::vector<int>();

// 添加元素到 vector
vec->push_back(10);
vec->push_back(20);
vec->push_back(30);

// 释放 vector
vec->~vector();  // 手动调用析构函数
```

## placement new

允许在已经分配好的内存区域中创建对象，可以决定对象存储的确切位置。

```cpp
std::vector<std::vector<float>>* data = new(v_mem) std::vector<std::vector<float>>();
```

其中 v_mem 为指针，指向预分配的内存地址。

## 保存已访问的数据

哈希表 unordered_set

```cpp
std::unordered_set<ListNode*> visited;

if (visited.count(p)) { ... }
visited.insert(p);
```

## 构造 vector：通过其他 STL 容器初始化一个 vector

```cpp
// eg.1
std::unordered_set<int> us;
// ...
return vector<int>(us.begin(), us.end());

// eg.2
int arr[5] = {1, 2, 3, 4, 5};
return vector<int>(arr, arr+5);
```

vector 构造函数：
```cpp
// 1
vector( const Allocator& = Allocator() );

// 2
vector( size_type n,constT& value = T(), const Allocator& = Allocator() );

// 3
// 上面例子中用的这个构造函数
// 利用迭代器
template <class InputIterator>  
vector ( InputIterator first, InputIterator last, const Allocator& = Allocator() );

// 4
vector ( const vector<T,Allocator>& x );
```

## 获取文件大小

```cpp
// 获取文件信息
int fd = open("...");
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
```

通过 `seekg()` 和 `tellg()`：

```cpp
std::ifstream in("...", std::ios::binary);
in.seekg(0, std::ios::end);
std::ios::pos_type ss = in.tellg();
size_t fsize = (size_t)ss;
```

## 多态中父类析构调用子类析构

将析构函数声明为虚函数实现。

```cpp
class A
{
public:
    explicit A() {}
    virtual ~A() { cout << "~A()" << endl; }
};

class B : public A
{
public:
    explicit B() {}
    virtual ~B() { cout << "~B()" << endl; }
};

A *a = new B();
a->~A();

// result
// print: ~B()
//        ~A()
```

## barrier 方式的无锁线程同步

`Linux` 中提供了多种同步机制，其中使用 `barrier` 是多线程之间进行同步的方法之一。

假设多个线程约定一个栅栏，只有当所有的线程都达到这个栅栏时，栅栏才会放行，否则到达此处的线程将被阻塞。

在代码中，当所有 `thread` 都执行到 `barrier_wait()` 后，才继续执行后续代码。

```cpp
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>

pthread_barrier_t barrier;

void* initor(void* args) {
	printf("---------------thread init work(%d)--------------\n", time(NULL));
	//模拟初始化工作。
	sleep(3);
	//到达栅栏
	pthread_barrier_wait(&barrier);
	printf("--------------thread start work(%d)--------------\n", time(NULL));
	sleep(3);
	printf("--------------thread stop work(%d)--------------\n", time(NULL));
	return NULL;
}

int main(int argc, char* argv[]) {
  	//初始化栅栏，该栅栏等待两个线程到达时放行
	pthread_barrier_init(&barrier, NULL, 2);
	printf("**************main thread barrier init done****************\n");
	pthread_t pid;
	pthread_create(&pid, NULL, &initor, NULL);
	printf("**************main waiting(%d)********************\n", time(NULL));
	//主线程到达，被阻塞，当初始化线程到达栅栏时才放行。
  	pthread_barrier_wait(&barrier);
    printf("***************main after barrier wait(%d)***************\n", time(NULL));
	pthread_barrier_destroy(&barrier);
	printf("***************main start to work(%d)****************\n", time(NULL));
	sleep(10);
	pthread_join(pid, NULL);
	printf("***************thread complete(%d)***************\n", time(NULL));

    system("pause");
	return 0;
}
```

输出结果：

```bash
**************main thread barrier init done****************
**************main waiting(1725351103)********************
---------------thread init work(1725351103)-------------- 
--------------thread start work(1725351106)--------------
***************main after barrier wait(1725351106)***************
***************main start to work(1725351106)****************    
--------------thread stop work(1725351109)--------------
***************thread complete(1725351116)***************
```

## Linux 宏 likely()、unlikely()

使用 `likely()`，执行 if 后面的语句的机会更大，使用 `unlikely()`，执行 else 后面的语句机会更大一些。

```c
if (likely(a>b)) {
　　fun1();
}
if (unlikely(a>b)){
　fun2();
}
```

这里就是程序员可以确定 `a>b` 在程序执行流程中出现的可能相比较大，因此运用了 `likely()` 告诉编译器将 `fun1()` 函数的二进制代码紧跟在前面程序的后面，这样就 cache 在预取数据时就可以将 `fun1()` 函数的二进制代码拿到 cache 中。这样，也就添加了 `cache` 的命中率。

同样的，`unlikely()` 的作用就是告诉编译器，`a<=b` 可能行大，`fun2()` 紧跟前面程序。

总之，`likely` 和 `unlikely` 的功能就是添加 cache 的命中率，提高系统执行速度。

## 条件编译与 CFLAGS

条件编译：

```c
#ifdef PERF_LATENCY_LOG
    uint32_t io_id;
    ...
    clock_gettime(...);
#endif
```

意思是，只有当在编译时宏 `PERF_LATENCY_LOG` 被定义了，`#ifdef ... #endif` 中的代码块才会被编译并执行；否则该代码块不会被编译执行。

这个宏可以在多个地方定义：

### 定义在头文件中

```c
// util.h
#define PERF_LATENCY_LOG
...
```

### 在 CMakeLists.txt 中

通过 `CFLAGS` 变量。

* 全局编译

```bash
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g")
```

* 区分编译

```bash
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG")

set(CMAKE_CXX_FLAGS_Release "${CMAKE_CXX_FLAGS_Release} -DNDBUG")
set(CMAKE_C_FLAGS_Release "${CMAKE_C_FLAGS_Release} -DNDBUG")
```

对于 `Debug` 模式，编译选项实际使用的值是 `CMAKE_CXX_FLAGS` 和 `CMAKE_CXX_FLAGS_DEBUG` 的值的组合（不管 `CMAKE_CXX_FLAGS_RELEASE` 设置什么值都不会被加入到编译选项中）。

对于 `Release` 模式，编译选项实际使用的值是 `CMAKE_CXX_FLAGS` 和 `CMAKE_CXX_FLAGS_RELEASE` 的值的组合（不管 `CMAKE_CXX_FLAGS_DEBUG` 设置什么值都不会被加入到编译选项中）。

```bash
mkdir debug
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

```bash
mkdir release
cd release
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

### 在 Makefile 中

通过 `CFLAGS` 变量。

```makefile
# Makefile
# 直接赋值或者追加
CFLAGS = -g -Wall
CFLAGS += -DPERF_LATENCY_LOG
```

### 通过 make 命令

```makefile
# Makefile
CFLAGS=$(CFLAG)
CFLAGS+=-g -Wall
...
```

输入 `make` 编译命令：

```bash
make CFLAG=-DPERF_LATENCY_LOG
```

### 在 configure 文件中

`configure` 文件本质可以看成是 `shell` 脚本文件，用来在 `make` 前配置 `make` 参数。

可以在 `configure` 文件中根据 `configure` 的参数来设置 `make` 的 `CFLAGS` 变量值。

这种情况一般是存在多级 `Makefile`，项目通过 `configure` 来配置。

```sh
# configure
...
# 添加 PERF_LATENCY_LOG
        --with-perf-latency-log)
            CONFIG[PERF_LATENCY_LOG]=y
            ;;
...

# CFLAGS 添加相应宏
if [[ "${CONFIG[PERF_LATENCY_LOG]}" = "y" ]]; then
    echo -n "Configuring PERF_LATENCY_LOG..."
    CFLAGS="${CFLAGS} -DPERF_LATENCY_LOG"
    echo "done."
fi

# CONFIG
# perf latency log
CONFIG_PERF_LATENCY_LOG=n
```

这样在 `./configure` 时，加上 `--with-perf-latency-log` 参数，就可以在 `CFLAGS` 变量中添加 `-DPERF_LATENCY_LOG` 宏定义，再输入 `make` 就不需要输入参数了。

## 局部变量块作用域

大括号 `{}` 在 C++ 中用于定义一个作用域。在这个作用域内，局部变量被创建并使用；当这个作用域结束时，局部变量会被销毁释放资源。

```cpp
{
    std::string string_val;
    // If it cannot pin the value, it copies the value to its internal buffer.
    // The intenral buffer could be set during construction.
    PinnableSlice pinnable_val(&string_val);
    db->Get(ReadOptions(), db->DefaultColumnFamily(), "key2", &pinnable_val);
    assert(pinnable_val == "value");
    // If the value is not pinned, the internal buffer must have the value.
    assert(pinnable_val.IsPinned() || string_val == "value");
}
// 在外面不能访问 string_val 变量
```

## std::async

以异步方式执行函数或者任务，并且能够在未来某个时间点获取执行结果。

```cpp
#include <future>
#include <iostream>

std::future<int> f1 = std::async(std::launch::async, [](){
        return 8; 
    });
 
cout<<f1.get()<<endl; //output: 8
 
std::future<int> f2 = std::async(std::launch::async, [](){
        cout<<8<<endl;
    });
 
f2.wait(); //output: 8
 
std::future<int> future = std::async(std::launch::async, [](){
        std::this_thread::sleep_for(std::chrono::seconds(3));
        return 8; 
    });
  
    std::cout << "waiting...\n";
    std::future_status status;
    do {
        status = future.wait_for(std::chrono::seconds(1));
        if (status == std::future_status::deferred) {
            std::cout << "deferred\n";
        } else if (status == std::future_status::timeout) {
            std::cout << "timeout\n";
        } else if (status == std::future_status::ready) {
            std::cout << "ready!\n";
        }
    } while (status != std::future_status::ready);
  
    std::cout << "result is " << future.get() << '\n';
```

## 