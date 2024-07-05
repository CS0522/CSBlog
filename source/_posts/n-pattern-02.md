---
title: 【学习笔记】设计模式（二）模板方法
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
date: 2024-07-03 00:54:57
---

设计模式之模板方法。

<!-- more -->

## 模板方法

定义一个操作中的算法的骨架（稳定），而将一些步骤（变化）放到子类中。模板方法使得子类不需要修改父类的结构（复用）即可重定义（override，重写）该算法中的某些特定步骤。


## 使用场景

一个算法骨架稳定，但其中一些步骤存在变化，可以使用模板方法。


## 举例

### 没有使用 template method

```cpp
class Library
{
    void step1() { ... }
    void step3() { ... }
    void step5() { ... }
};

class App
{
    void step2() { ... }
    void step4() { ... }
};

int main()
{
    Library l();
    App a();

    l.step1();
    a.step2();
    l.step3();
    a.step4();
    l.step5();
}
```

### 使用 template method

```cpp
class Library
{
    void step1() { ... }
    // 纯虚函数，子类实现
    virtual void step2() = 0;
    void step3() { ... }
    // 纯虚函数，子类实现
    virtual void step4() = 0;
    void step5() { ... }

    // 定义操作的算法骨架
    // 稳定 template method
    void run()
    {
        step1();
        // 虚函数的多态调用
        step2();
        step3();
        // 虚函数的多态调用
        step4();
        step5();
    }

    virtual ~Library() {}
};

class App : public Library
{
    virtual void step2() { ... }
    virtual void step4() { ... }
};

int main()
{
    // 声明类型为父类，实际类型为子类
    Library *l = new App();
    l->run();
}
```