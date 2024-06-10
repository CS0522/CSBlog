---
title: 【刷题日记】记录题型思路
tags:
  - C++
  - 刷题
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
comments: false
cover: false
date: 2024-06-09 21:06:05
---

记录下遇到的题型，可以首先往哪个方向去考虑。

<!-- more -->

---

## 数组

### 二分查找

* {% post_link p-array-erfen-l704 %}  

* {% post_link p-array-erfen-l34 %}  

* {% post_link p-array-erfen-l69 %}

#### 循环边界条件

通过区间来判定：

* `[left, right]`: while(left <= right), left = mid + 1, right = mid - 1

* `[left, right)`: while(left < right), left = mid + 1, right = mid

* `(left, right)`: while (left < right - 1), left = mid, right = mid


### 双指针

* {% post_link p-array-doubleP-l26 %}  

* {% post_link p-array-doubleP-l27 %}  

* {% post_link p-array-doubleP-l283 %}  

* {% post_link p-array-doubleP-l844 %}  

* {% post_link p-array-doubleP-l977 %}

* {% post_link p-array-moveWindow-l209 %}

* {% post_link p-array-moveWindow-l904 %}


#### 快慢指针

* 快指针：用于遍历，查找元素

* 慢指针：用于写入，更新元素。题目一般需要在这一步进行不一样的处理

#### 左右指针

* 注意从两边向中间指针移动的时候，最后的边界条件。可以看在最后一次处理的时候，数组的所有元素是否都已经处理完毕

#### 滑动窗口

* 与指针有点类似。窗口长度固定或者不固定。看作一个区间，就是不断的调节子序列的起始位置和终止位置，从而得出我们要想的结果
* 实现滑动窗口，主要确定如下三点：
  * 窗口内是什么？
  * 如何移动窗口的起始位置？
  * 如何移动窗口的结束位置？


### 螺旋矩阵

* {% post_link p-array-spiralMatrix-l59 %}

#### 直接模拟

* 注意循环条件，找到循环时同一的操作。