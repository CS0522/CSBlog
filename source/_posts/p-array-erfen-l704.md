---
title: 【刷题日记】数组-二分查找-L704-Easy
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
  - 数组 - 二分查找
comments: false
cover: false
date: 2024-04-25 19:58:50
---

开始刷题了。从基础开始重新学，记录一下重新学习的知识点以及做题的思路。跟着[《代码随想录》](https://www.programmercarl.com/)学习。

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

<!-- more -->

---

[704. 二分查找](https://leetcode.cn/problems/binary-search/description/)

## 思路

* nums 中无重复元素且有序，可以二分查找，结果唯一
* 当 left < right - 1 的时候循环，每次取 mid = (left + right) / 2 的值记为中点
* mid < target，则 target 在右边；否则在左边
* 结束条件为 left == right - 1，此时 left 和 right 紧挨着，这说明仍没有找到 target，因为如果找到了，while 中就 return 了


## 学习点

### 循环边界条件

通过区间来判定。

* `[left, right]`: while(left <= right), left = mid + 1, right = mid - 1
* `[left, right)`: while(left < right), left = mid + 1, right = mid
* `(left, right)`: while (left < right - 1), left = mid, right = mid


## 代码

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) 
    {
        int left = 0;
        int right = nums.size() - 1;
        // left bound
        if (nums[left] == target)
        {
            return left;
        }
        // right bound
        if (nums[right] == target)
        {
            return right;
        }
        // binary search
        while (left < right - 1)
        {
            int mid = (left + right) / 2;
            if (nums[mid] < target)
            {
                left = mid;
            }
            else if (nums[mid] > target)
            {
                right = mid;
            }
            else
            {
                return mid;
            }
        }
        // not found
        return -1;
    }
};
```