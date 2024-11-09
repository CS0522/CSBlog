---
title: 【学习笔记】设计模式（二十一）责任链
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
date: 2024-11-09 16:01:17
---

数据结构：责任链

<!-- more -->

## 模式类型

数据结构

## 责任链

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

每个检查步骤都可被抽取为仅有单个方法的类，并执行检查操作。请求及其数据则会被作为参数传递给该方法。

将这些处理者连成一条链。链上的每个处理者都有一个成员变量来保存对于下一处理者的引用。除了处理请求外，处理者还负责沿着链传递请求。请求会在链上移动，直至所有处理者都有机会对其进行处理。

处理者可以决定不再沿着链传递请求，这可高效地取消所有后续处理步骤。

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-pattern-21/chain.png)

## 使用场景

* 使用不同方式处理不同种类请求，请求顺序和类型预先未知
* 按顺序处理请求

## 举例

### 使用责任链

```cpp
class Request {

};

// ChainHandler 基类
class ChainHandler {
private:
    // 成员变量，保存了责任链
    ChainHandler *nextChain;

    void sendRequestToNextHandler(const Request &req)
    {
        if (!nextChain)
            nextChain->handle(req);
    }

protected:
    virtual bool canHandleRequest(const Request &req) = 0;
    virtual void processRequest(const Request &req) = 0;

public:
    ChainHandler() { nextChain = nullptr; }
    void setNextChain(ChainHandler *next)
    {
        nextChain = next;
    }
    void handle(const Request &req)
    {
        if (canHandleRequest(req))
            processRequest(req);
        else
            sendRequestToNextHandler(req);
    }
};

// 具体 Handler 类
class Handler1 : public ChainHandler {
    // 重写虚函数
    bool canHandleRequest() override { ... }
    void processRequest() override { ... }
};

class Handler2 : public ChainHandler {
    // 重写虚函数
    bool canHandleRequest() override { ... }
    void processRequest() override { ... }
};
```