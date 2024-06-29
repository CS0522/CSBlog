---
title: 【刷题日记】数组-长度最小的子数组-L209-Medium
tags:
  - C++
  - 刷题
  - 数组
  - 滑动窗口
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组 - 滑动窗口
comments: false
cover: false
date: 2024-06-10 12:09:55
---

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其总和大于等于 target 的长度最小的 
子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

<!-- more -->

---

## 思路

* 双指针，一个起点，一个末点，不必要的遍历次数较多
* 滑动窗口，当不满足条件时，end前进，扩大窗口；当满足条件后，start前进，缩小窗口，找到最小长度

## 学习点

* 第一次考虑滑动窗口的时候，思路是窗口长度定长，每次循环根据窗口长度，这样复杂度较高
* 动态长度的滑动窗口，满足条件后缩小窗口长度，减少很多遍历次数
* 原理就是，当满足条件后，以 `start` 开头的满足条件的子数组最小的必是这个数组，因此可以不需要比较之后的以 `start` 开头的子数组，所以移动 `start` 指针
* 实现滑动窗口，主要确定如下三点：
  * 窗口内是什么？
  * 如何移动窗口的起始位置？
  * 如何移动窗口的结束位置？

## 代码

```cpp
// 暴力解法
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();
        int minLen = n;
        bool found = false;
        // 快慢指针
        for (int slow = 0; slow < n; slow++)
        {
            int currSum = 0;
            for (int fast = slow; fast < n; fast++)
            {
                currSum += nums[fast];
                // 找到了
                if (currSum >= target)
                {
                    found = true;
                    if (minLen > (fast - slow + 1))
                        minLen = fast - slow + 1;
                }
            }
        }

        return (found ? minLen : 0);
    }
};
```

**update：滑动窗口**

```cpp
// 滑动窗口
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        // move window
        int n = nums.size();
        int minLen = n;
        int start = 0;
        int end = 0;
        int sum = 0;
        bool found = false;
        while (end < n && start <= end)
        {
            sum += nums[end];
            // 不满足条件，end前进
            if (sum < target)
            {
                end++;
            }
            // 满足条件，start前进，为了找到最小长度
            else
            {
                found = true;
                // 若更小，update minLen
                if ((end - start + 1) < minLen)
                    minLen = end - start + 1;
                sum -= nums[start] + nums[end];
                start++;
            }
        }

        return (found ? minLen : 0);
    }
};
```