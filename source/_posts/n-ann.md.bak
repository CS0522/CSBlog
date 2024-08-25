---
title: 【学习笔记】近似最近邻搜索（Approximate Nearest Neighbor Search）
tags:
  - ANN
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - ANN
comments: false
cover: false
date: 2024-05-15 20:17:11
---

向量检索、近似最近邻搜索的相关基础知识。

到后面不想写了...

<!-- more -->

---

## Overview

向量检索在二维或更高维空间中，常用距离方法包括：欧式距离、余弦距离、内积、投影...

KNN 问题中，精准找到 K 个最近邻往往代价很大，
当向量数量、维度特别大时，计算量不可估计。
因此实际应用中避免全量数据匹配。
近似最近邻（ANN），以更小的计算量获取更可靠的最近邻结果。
相应的各种 ANN 算法，都是通过牺牲一定的精度来换取时间和空间上的优势。

### 分类

ANN 算法常常被分为四个大类：

* 基于树结构
* 基于Hash
* 基于量化 (Quantization) 
* 基于图 (Graph) 

基于图的算法一般要优于其它三种类型的算法，表现在查询的速度和召回率上面。

在基于图的 ANN 算法中，如何构建图至关重要，也就是构建图中的各个边。


## 沃罗诺伊图 (Voronoi Diagram) 和德劳内图 (Delaunay Graph)

参考链接：

https://zhuanlan.zhihu.com/p/610454162

## 单调搜索网络 (Monotonic Search Network)

单调图 (Monotonic Graph) 或者单调搜索网络 (Monotonic Search Network)：

对于图 $G$ 上的一条路径 $V_{1}, V_{2}, V_{3}, ..., V_{n}$，
如果对任意 $i$，有 $Dist(V_{i}, V_{n}) > Dist(V_{i}, V_{n})$，
其中 $Dist()$ 表示两点之间的距离（实际距离），
那么这是一条单调路径。
如果对于图上的任意两点 $U$ 和 $V$，
总存在至少一条单调路径，那么 $G$ 是一个单调图。
单调图也被称为单调搜索网络，即 Monotonic Search Network。


## 单调相对邻域图 (Monotonic Relative Neighborhood Graph)

参考链接：

https://zhuanlan.zhihu.com/p/610454162

## 导航传播图 (Navigating Spreading-out Graph)

参考链接：

https://zhuanlan.zhihu.com/p/610454162