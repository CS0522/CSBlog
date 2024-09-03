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

## 