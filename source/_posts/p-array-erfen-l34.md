---
title: 【刷题日记】数组-在排序数组中查找元素的第一个和最后一个位置-L34-Medium
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
date: 2024-04-25 20:52:53
---

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。如果数组中不存在目标值 target，返回 [-1, -1]。你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。

<!-- more -->

---

## 思路

* 执行 2 次二分查找，第一次查找第一个位置，第二次查找第二个位置
* 第一次查找 first，mid 找到后，往左区间继续查找直到找到第一个位置 first，注意与普通二分查找不同的是找到后 while 循环也不结束，直到 left > right
* 第二次查找 last 同理，往右区间查找


## 学习点

注意二分查找的 while 循环边界条件：
* 之前解答中为 `while (left < right - 1)`，因为在循环中 `left = mid, right = mid`，而 `mid` 已经验证过不等于 `target`

* 这里为 `while (left <= right)`，因为在循环中 `left = mid + 1, right = mid - 1`，`left` 和 `right` 没有验证是否等于 `target`，所以在最后一次循环 `left = right` 时，验证 `mid(=left=right)` 位置是否等于 `target`


## 代码

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        
        // empty
        if (nums.size() == 0)
        {
            return vector<int> {-1, -1};
        }

        vector<int> res(2);
        int left = 0;
        int right = nums.size() - 1;
        // 提前赋值
        int first = -1;
        int last = -1;

        // 分两次查找
        // 查找第一个
        while (left <= right)
        {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target)
            {
                first = mid;
                // 往左边找，找到第一个位置
                right = mid - 1;
            }
            else if (nums[mid] < target)
            {
                left = mid + 1;
            }
            else
            {
                right = mid - 1;
            }
        }

        // 查找第二个
        left = 0;
        right = nums.size() - 1;
        while (left <= right)
        {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target)
            {
                last = mid;
                // 往右边找，找到最后一个位置
                left = mid + 1;
            }
            else if (nums[mid] < target)
            {
                left = mid + 1;
            }
            else
            {
                right = mid - 1;
            }
        }

        res[0] = first;
        res[1] = last;
        return res;
    }
};
```