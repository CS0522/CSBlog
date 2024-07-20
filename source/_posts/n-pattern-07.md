---
title: 【学习笔记】设计模式（七）工厂模式
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
date: 2024-07-08 16:49:33
---

设计模式之工厂模式。

<!-- more -->

## 工厂模式

通过对象创建的模式绕开 new，避免对象创建过程中导致的紧耦合（依赖具体类），从而支持对象创建的稳定。它是接口抽象后的第一步工作。

## 使用背景

定义一个创建对象的接口，让子类决定实例化哪一个类。

用于隔离类对象和使用者和具体类型之间的耦合关系。面对经常变化的具体类型，紧耦合关系（new）会导致软件的脆弱。

缺点在于要求创建方法、参数相同。

## 举例

### 未使用工厂模式

```cpp
class MainForm
{
    TextBox* textBox;
    // ...
    ProgressBar* progressBar;

    void button_click()
    {
        // ...
        ISplitter *splitter = new BinarySplitter( ... );
        splitter->split();
    }
};

class ISplitter
{
    virtual void split() = 0;
};

class BinarySplitter : public ISplitter
{
    virtual void split() { ... }
};

class TextSplitter : public ISplitter
{
    virtual void split() { ... }
};

class VideoSplitter : public ISplitter
{
    virtual void split() { ... }
};
```

### 使用工厂模式

```cpp
// 主体类
class MainForm
{
    TextBox* textBox;
    // ...
    ProgressBar* progressBar;
    SplitterFactory* factory;

    MainForm(SplitterFactory* factory)
    {
        this->factory = factory;
    }

    void button_click()
    {
        // ...
        ISplitter *splitter = factory->create_splitter();
        splitter->split();
    }
};

// 接口类
class ISplitter
{
    virtual void split() = 0;
};

// 工厂
class SplitterFactory
{
    virtual ISplitter* create_splitter() = 0;
};

// 具体类
class BinarySplitter : public ISplitter
{
    virtual void split() { ... }
};

class TextSplitter : public ISplitter
{
    virtual void split() { ... }
};

class VideoSplitter : public ISplitter
{
    virtual void split() { ... }
};

// 具体工厂
class BinarySplitterFactory : public SplitterFactory
{
    virtual ISplitter* create_splitter()
    {
        return new BinarySplitter( ... );
    }
};

class TextSplitterFactory : public SplitterFactory
{
    virtual ISplitter* create_splitter()
    {
        return new TextSplitter( ... );
    }
};

class VideoSplitterFactory : public SplitterFactory
{
    virtual ISplitter* create_splitter()
    {
        return new VideoSplitter( ... );
    }
};
```