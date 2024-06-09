---
title: 【刷题日记】数组-删除有序数组中的重复项-L29-Easy
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
date: 2024-06-09 12:35:11
---

给你一个 非严格递增排列 的数组 nums，请你原地删除重复出现的元素，使每个元素 只出现一次，返回删除后数组的新长度。元素的相对顺序应该保持一致。然后返回 nums 中唯一元素的个数。

<!-- more -->

---

## 思路

* 采用 27 题类似的思路快慢指针，并存储上一个比较的元素。如果和上一个相同，则 fast 指针前进，slow 指针不动
* 因为本题是非严格递增序列，可以采用如此思路。如果不是非严格递增、递减序列，该思路不可行

## 学习点

* 快慢指针

## 代码

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        // 快慢指针
        int slow = 1;
        // 存储上一个比较元素
        int temp = nums[0];
        for (int fast = 1; fast < nums.size();fast++)
        {
            // 非重复元素
            if (nums[fast] != temp)
            {
                nums[slow++] = nums[fast];
                temp = nums[fast];
            }
            // 重复元素
            // fast 指针前进
        }
        return slow;
    }
};
```

**update: 官方解法**

思路一致，优化在不需要 temp 额外存储

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int index = 1;
        int len = nums.size();
        for(int i = 1; i < len; i++)
        {
            if(nums[i] != nums[i - 1])
            {
                nums[index++] = nums[i];
            }
        }
        return index;
    }
};
```