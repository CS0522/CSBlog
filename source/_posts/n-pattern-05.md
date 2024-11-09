---
title: 【学习笔记】设计模式（五）装饰模式
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
date: 2024-07-04 21:04:31
---

单一职责：装饰模式。

<!-- more -->

## 模式类型

单一职责

## 装饰模式

某些时候“过度使用继承来扩展对象的功能”，导致缺乏灵活性；随着子类（扩展功能）的增多，各种子类的组合会导致更多子类的膨胀。

如何使“对象功能的扩展”根据需要来动态实现？避免子类的膨胀？

通过组合（装饰类包含指向主体类的基类的指针）而非继承的方法，装饰模式实现了在运行时动态扩展对象功能的能力。


## 使用场景

在已有的功能上添加一系列扩展操作。主体类在多个方向上的扩展功能。

## 举例

### 未使用装饰模式

其中的额外的扩展操作（加密操作）都是一致的。

```cpp
// 业务操作
class Stream
{
public:
    virtual char read(int number) = 0;
    // 定位文件流
    virtual void seek(int pos) = 0;
    virtual void write(char data) = 0;

    virtual ~Stream() { }
};

// 主体类
// 文件流
class FileStream : public Stream
{
public:
    virtual char read(int number)
    {
        // ...
    }
    virtual void seek(int pos)
    {
        // ...
    }
    virtual void write(char data)
    {
        // ...
    }
};

// 网络流
class NetworkStream : public Stream
{
    // ...
};

// 内存流
class MemoryStream : public Stream
{
    // ...
};

// 扩展操作
class CryptoFileStream : public FileStream
{
public:
    virtual void read(int number)
    {
        // ...加密操作
        FileStream::read(number);
    }
    virtual void seek(int pos)
    {
        // ...加密操作
        FileStream::seek(pos);
    }
    virtual void write(char data)
    {
        // ...加密操作
        FileStream::write(data);
    }
};
```

当额外的操作需要修改时，就需要大量的修改工作量。


### 使用装饰模式

```cpp
// 业务操作
class Stream
{
public:
    virtual char read(int number) = 0;
    // 定位文件流
    virtual void seek(int pos) = 0;
    virtual void write(char data) = 0;

    virtual ~Stream() { }
};

// 主体类
// 文件流
class FileStream : public Stream
{
public:
    virtual char read(int number)
    {
        // ...
    }
    virtual void seek(int pos)
    {
        // ...
    }
    virtual void write(char data)
    {
        // ...
    }
};

// 网络流
class NetworkStream : public Stream
{
    // ...
};

// 内存流
class MemoryStream : public Stream
{
    // ...
};

// 扩展操作
// 装饰基类
class DecoratorStream : public Stream
{
protected:
    Stream* stream;
public:
    DecoratorStream(Stream* stream): stream(stream) { }
}; 

// 装饰类1
class CryptoStream : public DecoratorStream
{
public:
    // 给 stream 赋值，是文件流、网络流还是内存流
    CryptoStream(Stream* stream): DecoratorStream(stream) { }
 
    virtual void read(int number)
    {
        // ...加密操作
        stream->read(number);
    }
    virtual void seek(int pos)
    {
        // ...加密操作
        stream->seek(pos);
    }
    virtual void write(char data)
    {
        // ...加密操作
        stream->write(data);
    }
};

// 装饰类2
class BufferStream : public DecoratorStream
{
    Stream* stream;
public:
    // 给 stream 赋值，是文件流、网络流还是内存流
    BufferStream(Stream* stream): DecoratorStream(stream) { }
    // ...
};


void process()
{
    FileStream *fs = new FileStream();
    CryptoStream * cs = new CryptoStream(fs);
    BufferStream * bs = new BufferStream(fs);
}
```