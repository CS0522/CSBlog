---
title: 【学习笔记】设计模式（十九）组合模式
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
date: 2024-11-09 11:47:09
---

数据结构：组合模式

<!-- more -->

## 模式类型

数据结构

## 组合模式

将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

简化树形结构中对象的处理，无论它们是单个对象还是组合对象。

解耦客户端代码与复杂元素的内部结构，使得客户端可以统一处理所有类型的节点。

## 使用场景

* 需要表示对象的层次结构，如文件系统或组织结构

* 图形界面

## 举例

### 使用组合模式

```cpp
class Component {
protected:
    Component *parent_;
public:
    virtual ~Component() {}
    void SetParent(Component *parent) {
        this->parent_ = parent;
    }
    Component *GetParent() const {
        return this->parent_;
    }

    virtual void Add(Component *component) {}
    virtual void Remove(Component *component) {}

    virtual bool IsComposite() const {
      return false;
    }

    virtual std::string Operation() const = 0;
};

// 树叶
// 执行具体操作
class Leaf : public Component {
public:
    std::string Operation() const override {
        return "Leaf";
    }
};

// 树枝
class Composite : public Component {
protected:
    std::list<Component *> children_;

public:
    void Add(Component *component) override {
        this->children_.push_back(component);
        component->SetParent(this);
    }

    void Remove(Component *component) override {
        children_.remove(component);
        component->SetParent(nullptr);
    }
    bool IsComposite() const override {
        return true;
    }

    std::string Operation() const override {
        std::string result;
        for (const Component *c : children_) {
            if (c == children_.back()) {
                result += c->Operation();
            } else {
                result += c->Operation() + "+";
            }
        }
        return "Branch(" + result + ")";
    }
};
```