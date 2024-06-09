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

## 思路

### 二分查找

* {% post_link p-array-erfen-l704 %}  

* {% post_link p-array-erfen-l34 %}  

* {% post_link p-array-erfen-l69 %}


#### 循环边界条件

通过区间来判定：

* `[left, right]`: while(left <= right), left = mid + 1, right = mid - 1

* `[left, right)`: while(left < right), left = mid + 1, right = mid

* `(left, right)`: while (left < right - 1), left = mid, right = mid


### 移除元素

* {% post_link p-array-removeEle-l26 %}  

* {% post_link p-array-removeEle-l27 %}  

* {% post_link p-array-removeEle-l283 %}  

* {% post_link p-array-removeEle-l844 %}  

* {% post_link p-array-removeEle-l977 %}

#### 快慢指针

* 快指针：用于遍历，查找元素

* 慢指针：用于写入，更新元素。题目一般需要在这一步进行不一样的处理

#### 左右指针

* 注意从两边向中间指针移动的时候，注意最后的边界条件。可以看在最后一次处理的时候，数组的所有元素是否都已经处理完毕。