---
title: 【学习笔记】设计模式（二十二）命令模式
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
date: 2024-11-09 17:04:02
---

行为变化：命令模式

<!-- more -->

## 模式类型

行为变化

## 命令模式

将请求转换为一个包含与请求相关的所有信息的独立对象。该转换让你能根据不同的请求将方法参数化、延迟请求执行或将其放入队列中，且能实现可撤销操作。

## 使用场景

* 通过操作来参数化对象

* 用于代替包含行为的参数化 UI 元素的回调函数，此外还被用于对任务进行排序和记录操作历史记录等。


## 举例

### 使用命令模式

```cpp
class Command {
public:
    virtual void execute() = 0;
};

class ConcreteCommand1 : public Command {
private:
    string arg;
public:
    void execute() override {
        // ...
    }
};

class ConcreteCommand2 : public Command {
private:
    string arg;
public:
    void execute() override {
        // ...
    }
};

class ConcreteCommands : public Command {
private:
    vector<Command> commands;
public:
    void addCommand() { ... }
    void execute() override {
        for command in commands:
            // ...
    }
}

class Invoker {
private:
    Command *comm;
public:
    void Invoker(const Command *comm_) : comm(comm) { }
    void doSomething() {
        this->comm->execute();
    }
};
```