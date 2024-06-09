---
title: 【刷题日记】数组-有序数组的平方-L977-Easy
tags:
  - C++
  - 刷题
  - 数组
  - 移除元素
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组
  - 移除元素
comments: false
cover: false
date: 2024-06-09 22:22:50
---

给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。

<!-- more -->

---

## 思路

* 左右指针
* 注意`非递减排序`条件，存在负数，那么可能中间存在 0，他的左边从左到右为递减，他的右边从右到左为递减，可以采用左右指针，类似于归并的思路进行排序

## 学习点

* 左右指针
* 注意边界条件！！

## 代码

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        // **非递减顺序**
        int len = nums.size();
        vector<int> res(len);

        // 左右指针
        int left = 0;
        int right = len - 1;
        int pos = len - 1;
        // left <= right，这样才是写入了 5 次数据
        // 如果是 left < right，
        // 这时候循环内最后加入的数只能是 left 或者 right
        // 有一个还没有被处理
        for (; left <= right; )
        {
            int left_square = nums[left] * nums[left];
            int right_square = nums[right] * nums[right];
            if (left_square > right_square)
            {
                res[pos] = left_square;
                left++;
            }
            else
            {
                res[pos] = right_square;
                right--;
            }
            pos--;
        }

        return res;
    }
};
```