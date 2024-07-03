---
title: 【学习笔记】设计模式（一）简介与设计原则
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
date: 2024-06-29 17:10:29
---

开始学习设计模式。

<!-- more -->

## 简介

* **可复用**面向对象软件基础

* 好的软件设计：**复用！！！**

* **接口标准化！！！**

* 设计模式的前提是存在一个**稳定点**

* 在变化和稳定之间寻找稳定点

* 要看出代码哪部分稳定，哪部分变化


## 面向对象设计原则

### 依赖倒置原则 DIP

* 高层模块（稳定）不应该依赖于低层模块（变化），二者都应该依赖于抽象（稳定）。

* 实现细节（变化）应该依赖于抽象（稳定）。

即从问题中提出抽象类，找到低层稳定的抽象。

比如初始 MainForm 依赖于 Line 和 Rectangle，但低层模块这两个变化，实现细节不一致，因此还需要提出这两个的抽象类 Shape，因此修改后，MainForm 依赖于 Shape，Line 和 Rectangle 依赖于 Shape。


### 开放封闭原则 OCP

* 对扩展开放，对更改封闭。

* 类模块可扩展，但是不可修改。


### 单一责任原则 SRP

* 一个类应该仅有一个引起它变化的原因。

* 变化的方向隐含着类的责任。


### Liskov 替换原则 LSP

* 子类能够替换他们的基类。


### 接口隔离原则 ISP

* 接口应该小而完备。

仅暴露有必要的方法，非必要不暴露。


### 优先使用对象组合，而不是继承

* 继承在某种程度上破坏了封装性，子类和父类耦合度高。

* 对象组合则要求被组合的对象具有良好定义的接口，耦合度较低。


### 封装变化点

* 使用封装来创建对象之间的分界层，在一侧进行修改，另一层不受影响，从而实现层次间的松耦合。


### 针对接口编程

* 不将（业务类）变量声明为某个特定的具体类，而是声明为接口类。

* 客户程序只需要知道对象的接口。

* 减少系统各部分的依赖关系，实现“高内聚、低耦合”。