---
title: 【学习笔记】设计模式（二十）迭代器
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
date: 2024-11-09 15:36:43
---

数据结构：迭代器

<!-- more -->

## 模式类型

数据结构

## 迭代器模式

将特定的数据结构封装在内部，在外部提供统一的接口，来实现与数据结构无关的访问。

数据结构内部变化各异，但希望在不暴露内部结构的同时，让外部客户“透明遍历”访问其中的元素。

## 使用场景

* C++ STL 迭代器

## 举例

### 面向对象的迭代器

运行时多态：

```cpp
template<typename T>
class Iterator {
public:
    virtual void first() = 0;
    virtual void next() = 0;
    virtual T& current() = 0;
};

template<typename T>
class MyCollection {
public:
    Iterator<T> createIterator() {
        return (new CollectionIterator<T>(this))
    }
};

template<typenmae T>
class CollectionIterator : public Iterator<T> {
    MyCollection<T> mc;

public:
    CollectionIterator(const MyCollection<T> &c): mc(c) {

    }

    void first() override {

    }
    void next() override {

    }
    T& current() override {

    }
};

void MyAlgorithm() {
    MyCollection<int> mc;
    Iterator<int> iter = mc.createIterator();
    for (iter.first(); ...) {
        // ...
    }
}
```