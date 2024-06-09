---
title: 【刷题日记】数组-x的平方根-L69-Easy
tags:
  - C++
  - 刷题
  - 数组
  - 二分查找
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组
  - 二分查找
comments: false
cover: false
date: 2024-04-25 20:42:46
---

给你一个非负整数 x，计算并返回 x 的算术平方根。由于返回类型是整数，结果只保留整数部分，小数部分将被舍去。

<!-- more -->

---

## 思路

官方解答思路：
* 只要mid <= x/mid，left左边界就会一直向右移动，ans 就会一直更新（变大），直到不在满足 mid <= x/mid 的条件为止，ans 就会停止更新，永远停在满足 mid<=x/mid 条件下面的最后一次更新，即满足 ans * ans <= mid 的最大值
* 为什么要加上=的判断条件，因为将左指针和右指针相同的时候，就是判断指针本身和目标值是否相等。每当这里想不清楚的时候，就拿最简单的例子：1，2，3来做推理。要想知道3在不在他们中间，先比较3和2，然后3比2大，left = mid + 1 = 3，right = right = 3；这时候left 和 right相等，他们的中间也就是3,3=3，于是3在他们中间，如果left < right 就返回，就会返回错误的答案。

## 代码

```cpp
class Solution {
public:
    int mySqrt(int x) {
        int low = 0;
        int high = x;
        int ans = -1;
        while (low <= high)
        {
            long mid = (low + high) / 2;
            long mid_square = mid * mid;
            if (mid_square <= x)
            {   
                ans = mid;
                low = mid + 1;
            }
            else
            {
                high = mid - 1;
            }
        }
        return ans;
    }
};
```