---
title: 【学习笔记】设计模式（十一）单例模式
tags:
  - 设计模式
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 设计模式
comments: false
cover: false
date: 2024-11-08 19:48:22
---

对象性能：单例模式。

<!-- more -->

## 模式类型

对象性能

## 单例模式

可能会存在一种特殊的类，需要保证他们在系统中只能存在一个实例，才能确保逻辑的正确性。

单例模式确保一个类只有一个实例，并提供了一个全局访问点来访问该实例。

单例类只能有一个实例，必须自己创建自己的唯一实例，且必须给所有其他对象提供这一实例。

## 使用场景

* 多线程多进程

* 生成唯一序列号

* WEB 中的计数器，避免每次刷新都在数据库中增加计数，先缓存起来

* 创建消耗资源过多的对象，如 I/O 与数据库连接等

## 举例

### 单例模式

1. 构造函数私有化；
2. 静态成员指针变量；
3. 提供 `static` 公有的 `getInstance()` 函数；

```cpp
class Singleton {
private: 
    // 构造函数私有化
    Singleton() {}
public:
    // 静态成员指针变量
    static Singleton* m_instance;
    // 公有 get 函数
    static Singleton* getInstance();
};

Singleton::m_instance = nullptr;

// 单线程版
Singleton* Singleton::getInstance() {
    if (!m_instance)
        m_instance = new Singleton();
    return m_instance;
}

// 多线程锁
Singleton* Singleton::getInstance() {
    Lock lock;
    if (!m_instance)
        m_instance = new Singleton();
    return m_instance;
}

// 多线程双检查锁
// 注意 C++ 内存序问题
// memory reorder
// 编译器的优化，导致指令乱序执行
Singleton* Singleton::getInstance() {
    if (!m_instance)
    {
        Lock lock;
        if (!m_instance)
            m_instance = new Singleton();
    }
    return m_instance;
}

// 变量添加 volatile 关键字
```