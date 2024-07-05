---
title: 【学习笔记】设计模式（六）桥模式
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
date: 2024-07-04 22:04:56
---

设计模式之桥模式。

<!-- more -->

## 桥模式

由于某些类型的固有实现逻辑，使他具有多个维度的变化。

将抽象部分（业务功能）与实现部分（平台实现）分离，使他们都可以独立地变化。


## 使用场景

与装饰模式类似，不过接口进行了拆分，将业务功能和平台实现进行了接口的拆分。

桥模式中的桥可以看作是 `Messager` 基类中的那个指向 `MessagerImp` 的指针，将二者组合起来，而不是继承。

以下例子为一个具有一个维度的变化。如果具有多个维度的变化，则需要多个指向基类的指针来充当“桥”组合各个基类。

与装饰模式作为对比，桥模式是主要区别是进行了接口的拆分。


## 举例

### 不使用桥模式

```cpp
class Messager
{
public:
    virtual void login() = 0;
    virtual void send_message() = 0;
    virtual void send_picture() = 0;

    virtual void play_sound() = 0;
    virtual void draw_shape() = 0;
    virtual void write_text() = 0;
    virtual void connect() = 0;

    virtual ~Messager() {}
};

// PC 平台
class PCMessagerBase : public Messager
{
    virtual void play_sound() { ... }
    virtual void draw_shape() { ... }
    virtual void write_text() { ... }
    virtual void connect() { ... }
};

// 移动平台
class MobileMessagerBase: public Messager
{
    virtual void play_sound() { ... }
    virtual void draw_shape() { ... }
    virtual void write_text() { ... }
    virtual void connect() { ... }
};

// PC 平台添加功能
class PCMessagerLite : public PCMessagerBase
{
    virtual void login()
    {
        PCMessagerBase::connect();
        // ...
    }
    virtual void send_message()
    {
        PCMessagerBase::write_text();
        // ...
    }
    virtual void send_picture()
    {
        PCMessagerBase::play_sound();
        PCMessagerBase::draw_shape();
        // ...
    }
}

// 移动平台添加功能
class MobileMessagerLite : public MobileMessagerBase
{
    virtual void login()
    {
        MobileMessagerBase::connect();
        // ...
    }
    virtual void send_message()
    {
        MobileMessagerBase::write_text();
        // ...
    }
    virtual void send_picture()
    {
        MobileMessagerBase::play_sound();
        MobileMessagerBase::draw_shape();
        // ...
    }
}

void process()
{
    MobileMessagerLite *m = new MobileMessagerLite();
}
```

### 使用桥模式

```cpp
// 对原本的 Messager 接口隔离
class Messager
{
    // 组合，而不是继承
    MessagerImp* messagerImp;
public:
    Messager(Messager* m): messagerImp(m) { }

    virtual void login() = 0;
    virtual void send_message() = 0;
    virtual void send_picture() = 0;

    virtual ~Messager() {}
};

class MessagerImp
{
public:
    virtual void play_sound() = 0;
    virtual void draw_shape() = 0;
    virtual void write_text() = 0;
    virtual void connect() = 0;

    virtual ~MessagerImp() {}
};

// PC 平台
class PCMessagerImp : public MessagerImp
{
    virtual void play_sound() { ... }
    virtual void draw_shape() { ... }
    virtual void write_text() { ... }
    virtual void connect() { ... }
};

// 移动平台
class MobileMessagerImp: public MessagerImp
{
    virtual void play_sound() { ... }
    virtual void draw_shape() { ... }
    virtual void write_text() { ... }
    virtual void connect() { ... }
};

// PC 平台、移动平台添加功能
// 添加的功能一致。因此和装饰模式一样
class MessagerLite : public Messager
{
    // 初始化
    MessagerLite(MessagerImp* m): messagerImp(m) { }

    virtual void login()
    {
        messagerImp->connect();
        // ...
    }
    virtual void send_message()
    {
        messagerImp->write_text();
        // ...
    }
    virtual void send_picture()
    {
        messagerImp->play_sound();
        messagerImp->draw_shape();
        // ...
    }
};


void process()
{
    MessagerImp *pcMessagerImp = new PCMessagerImp();
    MessagerLite *ml = new MessagerLite(pcMessagerImp);
}
```