---
title: 【刷题日记】哈希表-四数之和-L18-Medium
tags:
  - C++
  - 刷题
  - 哈希表
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 哈希表
comments: false
cover: false
date: 2024-08-12 00:34:36
---

给你一个由 n 个整数组成的数组 nums ，和一个目标值 target 。请你找出并返回满足下述全部条件且不重复的四元组 [nums[a], nums[b], nums[c], nums[d]] （若两个四元组元素一一对应，则认为两个四元组重复）：

<!-- more -->

---

## 思路

* 初步想法，类比与三数之和，使用双指针做法做。

## 学习点



## 代码

双指针法，需要考虑到四数之和可能会超过 `int` 范围：

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        // 双指针
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());
        if (nums.size() < 4)
            return res;
        
        // nums[i] + nums[j] + nums[left] + nums[right]
        size_t nums_size = nums.size();
        for (size_t i = 0; i < nums_size; i++)
        {
            // 对 a 去重
            if (i >= 1 && nums[i] == nums[i - 1])
                continue;
            for (size_t j = i + 1; j < nums_size; j++)
            {
                // 对 b 去重
                if (j >= i + 2 && nums[j] == nums[j - 1])
                    continue;
                
                int left = j + 1;
                int right = nums_size - 1;
                while (left < right)
                {
                    while (left < right && 
                            (long)nums[i] + (long)nums[j] + (long)nums[left] + (long)nums[right] > target)
                        --right;
                    while (left < right &&
                            (long)nums[i] + (long)nums[j] + (long)nums[left] + (long)nums[right] < target)
                        ++left;
                    // 如果不满足条件
                    if (left >= right)
                        break;
                    // 满足条件
                    else if ((long)nums[i] + (long)nums[j] + (long)nums[left] + (long)nums[right] == target)
                    {
                        res.emplace_back(vector<int>{nums[i], nums[j], nums[left], nums[right]});
                        // 指针前进去重
                        ++left;
                        while (left < right && nums[left] == nums[left - 1])
                            ++left;
                        --right;
                        while (left < right && nums[right] == nums[right + 1])
                            --right;
                    }
                }
            }
        }

        return res;
    }
};
```

优化的解法：

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        for (int k = 0; k < nums.size(); k++) {
            // 剪枝处理
            if (nums[k] > target && nums[k] >= 0) {
            	break; // 这里使用break，统一通过最后的return返回
            }
            // 对nums[k]去重
            if (k > 0 && nums[k] == nums[k - 1]) {
                continue;
            }
            for (int i = k + 1; i < nums.size(); i++) {
                // 2级剪枝处理
                if (nums[k] + nums[i] > target && nums[k] + nums[i] >= 0) {
                    break;
                }

                // 对nums[i]去重
                if (i > k + 1 && nums[i] == nums[i - 1]) {
                    continue;
                }
                int left = i + 1;
                int right = nums.size() - 1;
                while (right > left) {
                    // nums[k] + nums[i] + nums[left] + nums[right] > target 会溢出
                    if ((long) nums[k] + nums[i] + nums[left] + nums[right] > target) {
                        right--;
                    // nums[k] + nums[i] + nums[left] + nums[right] < target 会溢出
                    } else if ((long) nums[k] + nums[i] + nums[left] + nums[right]  < target) {
                        left++;
                    } else {
                        result.push_back(vector<int>{nums[k], nums[i], nums[left], nums[right]});
                        // 对nums[left]和nums[right]去重
                        while (right > left && nums[right] == nums[right - 1]) right--;
                        while (right > left && nums[left] == nums[left + 1]) left++;

                        // 找到答案时，双指针同时收缩
                        right--;
                        left++;
                    }
                }

            }
        }
        return result;
    }
};
```