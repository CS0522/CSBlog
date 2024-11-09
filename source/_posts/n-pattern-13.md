---
title: 【学习笔记】设计模式（十三）门面模式
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
date: 2024-11-08 20:36:11
---

接口隔离：门面模式

<!-- more -->

## 模式类型

接口隔离

## 门面模式

某些接口之间添加一层间接（稳定）接口，隔离本来紧密相互关联的接口。

简化外部程序与内部系统间的交互接口，将外部程序与内部系统的依赖相互解耦。

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-pattern-13/facade.png)

内部组件应该是“相互耦合关系较大的一系列组件”。

## 使用场景

为子系统中的一组接口提供一个一致（稳定）的界面。定义一个高层接口，使得子系统更加容易使用（复用）。

## 举例

* 电脑