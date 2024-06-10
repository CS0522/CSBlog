---
title: 【刷题日记】数组-移除元素-L27-Easy
tags:
  - C++
  - 刷题
  - 数组
  - 双指针
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组
  - 双指针
comments: false
cover: false
date: 2024-06-09 11:17:18
---

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素。元素的顺序可能发生改变。然后返回 nums 中与 val 不同的元素的数量。

<!-- more -->

---

## 思路

* 使用左右双指针，两个指针初始时分别位于数组的首尾，向中间移动遍历该序列

**update: 快慢指针**

* 通过一个快指针和慢指针在一个 for 循环下完成两个 for 循环的工作
  * 快指针：寻找新数组的元素，新数组就是不含有目标元素的数组
  * 慢指针：指向更新 新数组下标的位置

## 学习点

* 右指针 `right` 初始为 `nums.size() - 1`，但此时 `right` 并没有被遍历（被处理），因此完全遍历 `nums` 数组的条件为 `left <= right`，此时 `left` 指针会处理 `right`
* 可能会存在 `nums[right] == val`，因此在赋值前找到合适的 `right` 位置，减少赋值操作

**update: 快慢指针**

* slow 指针用于写入，fast 指针用于查找
* 当 `val == fast`，慢指针不动，快指针前进；当 `val != fast`，快慢指针同时前进


## 代码

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        // val 是要删除的元素
        // 左右指针
        int left = 0;
        int right = nums.size() - 1;
        while (left <= right)
        {
            if (nums[left] == val)
            {
                // 可能会存在 nums[right] == val
                while (nums[right] == val)
                {
                    right--;
                    if (right < left)
                    {
                        return left;
                    }
                }
                // 查找 right 合适位置
                nums[left] = nums[right];
                right--;
            }
            else
            {
                left++;
            }
        }
        return left;
    }
};
```

**update: 快慢指针**

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        // val 是要删除的元素
        // 快慢指针
        // slow 用于写入
        // fast 用于查找
        int slow = 0;
        int fast = 0;
        for (; fast < nums.size(); fast++)
        {
            // 当 val != fast，慢指针写入快指针数值
            // 快慢指针同时前进
            if (nums[fast] != val)
            {
                nums[slow++] = nums[fast];
            }
            // 当 al == fast，慢指针不写入
            // 慢指针不动，快指针前进
        }
        return slow;
    }
};
```