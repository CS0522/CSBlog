---
title: 【学习笔记】设计模式（十四）代理模式
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
date: 2024-11-09 11:07:21
---

接口隔离：代理模式

<!-- more -->

## 模式类型

接口隔离

## 代理模式

为其他对象提供一种代理以控制（隔离，使用接口）对这个对象的访问。

在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

代理必须和真实对象使用相同接口，才能伪装成服务对象。

代理模式建议新建一个与原服务对象接口相同的代理类， 然后更新应用以将代理对象传递给所有原始对象客户端。 代理类接收到客户端请求后会创建实际的服务对象， 并将所有工作委派给它。

如果需要在类的主要业务逻辑前后执行一些工作， 你无需修改类就能完成这项工作。 由于代理实现的接口与原类相同， 因此你可将其传递给任何一个使用实际服务对象的客户端。

## 使用场景

* 想在访问一个类时做一些控制
* 延迟初始化
* 远程代理
* 缓存代理
* 记录日志请求

## 举例

### 使用代理模式

```cpp
// 抽象接口
class Subject {
public:
    virtual void Request() const = 0;
};

// 真实对象
class RealSubject : public Subject {
public:
    void Request() const override {
        std::cout << "RealSubject: Handling request.\n";
    }
};

// 代理类
class Proxy : public Subject {
private:
    RealSubject *real_subject_;

    bool CheckAccess() const {
        // Some real checks should go here.
        std::cout << "Proxy: Checking access prior to firing a real request.\n";
        return true;
    }
    
    void LogAccess() const {
        std::cout << "Proxy: Logging the time of request.\n";
    }

public:
    Proxy(RealSubject *real_subject) : real_subject_(new RealSubject(*real_subject)) {
    }

    // 与真实对象实现相同的接口
    void Request() const override {
        if (this->CheckAccess()) {
            this->real_subject_->Request();
            this->LogAccess();
        }
    }
};

// client 端执行
void ClientCode(const Subject &subject) {
    // ...
    subject.Request();
    // ...
}
```