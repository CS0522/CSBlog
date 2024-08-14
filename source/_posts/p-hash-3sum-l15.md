---
title: 【刷题日记】哈希表-三数之和-L15-Medium
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
date: 2024-07-25 12:23:20
---

给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。

<!-- more -->

---

## 思路

* 刚开始用哈希表做，没做出来，去重不好处理。

* 考虑用双指针法做。

## 学习点

* 去重逻辑。

## 代码

我的双指针法，在去重处理上代码写的冗余：

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        // 双指针
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        if (nums[0] > 0 || nums[nums.size() - 1] < 0)
            return result;
        // a = nums[i], b = nums[left], c = nums[right]
        for (int i = 0; i < nums.size(); ++i)
        {
            // 对 a 去重
            if (i > 0 && nums[i] == nums[i - 1])
                continue;
            // b + c
            int left = i + 1;
            int right = nums.size() - 1;
            // i != left != right
            while (left < right)
            {
                while (nums[i] + nums[left] + nums[right] > 0)
                {
                    --right;
                    if (right <= i)
                        break;
                }
                while (nums[i] + nums[left] + nums[right] < 0)
                {
                    ++left;
                    if (left >= right)
                        break;
                }
                if (left >= right || right <= i)
                    break;
                // 满足条件
                else if (left < right && nums[i] + nums[left] + nums[right] == 0)
                {
                    result.emplace_back(vector<int>{nums[i], nums[left], nums[right]});
                    // 指针前进，并去重
                    ++left;
                    while (nums[left] == nums[left - 1])
                    {
                        ++left;
                        if (left >= right)
                            break;
                    }
                    --right;
                    while (nums[right] == nums[right + 1])
                    {
                        --right;
                        if (right <= i)
                            break;
                    }
                }
            }
        }
        return result;
    }
};
```

优化的双指针法：

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        // 找出a + b + c = 0
        // a = nums[i], b = nums[left], c = nums[right]
        for (int i = 0; i < nums.size(); i++) {
            // 排序之后如果第一个元素已经大于零，那么无论如何组合都不可能凑成三元组，直接返回结果就可以了
            if (nums[i] > 0) {
                return result;
            }
            // 去重a
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            int left = i + 1;
            int right = nums.size() - 1;
            while (right > left) {
                if (nums[i] + nums[left] + nums[right] > 0) right--;
                else if (nums[i] + nums[left] + nums[right] < 0) left++;
                else {
                    result.push_back(vector<int>{nums[i], nums[left], nums[right]});
                    // 去重逻辑应该放在找到一个三元组之后，对b 和 c去重
                    while (right > left && nums[right] == nums[right - 1]) right--;
                    while (right > left && nums[left] == nums[left + 1]) left++;

                    // 找到答案时，双指针同时收缩
                    right--;
                    left++;
                }
            }

        }
        return result;
    }
};
```

哈希法：

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        // 找出a + b + c = 0
        // a = nums[i], b = nums[j], c = -(a + b)
        for (int i = 0; i < nums.size(); i++) {
            // 排序之后如果第一个元素已经大于零，那么不可能凑成三元组
            if (nums[i] > 0) {
                break;
            }
            if (i > 0 && nums[i] == nums[i - 1]) { //三元组元素a去重
                continue;
            }
            unordered_set<int> set;
            for (int j = i + 1; j < nums.size(); j++) {
                if (j > i + 2
                        && nums[j] == nums[j-1]
                        && nums[j-1] == nums[j-2]) { // 三元组元素b去重
                    continue;
                }
                int c = 0 - (nums[i] + nums[j]);
                if (set.find(c) != set.end()) {
                    result.push_back({nums[i], nums[j], c});
                    set.erase(c);// 三元组元素c去重
                } else {
                    set.insert(nums[j]);
                }
            }
        }
        return result;
    }
};
```