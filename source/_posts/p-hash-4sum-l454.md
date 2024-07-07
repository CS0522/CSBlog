---
title: 【刷题日记】哈希表-四数相加II-L454-Medium
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
date: 2024-07-07 21:48:59
---

给你四个整数数组 nums1、nums2、nums3 和 nums4 ，数组长度都是 n ，请你计算有多少个元组 (i, j, k, l) 能满足：

0 <= i, j, k, l < n

nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0

<!-- more -->

---

## 思路

* 与 `L1. 两数之和` 思路类似，寻找 `target - x`。为了降低时间复杂度，因此将 `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0` 分组为 `nums1[i] + nums2[j] = 0 - nums3[k] - nums4[l]`，先对 `nums1 + nums2` 遍历，并存储已出现的元素和以及其个数；然后遍历 `nums3 + nums4`，并查找当前元素和的相反数以及其出现个数。

## 学习点

* 借鉴 `L1. 两数之和` 的思路，进行 `分组 + 哈希表`。

## 代码

修改前：（超出时间限制）

```cpp
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        // nums1 + nums2 = - nums3 - nums4
        multiset<int> ms;
        int res = 0;
        int nums_size = nums1.size();
        // 先遍历 nums1，nums2，保存所有可能取值
        for (int i = 0; i < nums_size; ++i)
        {
            for (int j = 0; j < nums_size; ++j)
            {
                ms.insert(nums1[i] + nums2[j]);
            }
        }
        // 后遍历 nums3，nums4，查找哈希集合中是否有其相反数
        for (int i = 0; i < nums_size; ++i)
        {
            for (int j = 0; j < nums_size; ++j)
            {
                int opp_count = ms.count(0 - nums3[i] - nums4[j]);
                // 如果找到了，并且有可能不止一个
                if (opp_count)
                    res += opp_count;
            }
        }

        return res;
    }
};
```

修改后代码，没有超过时间限制，可能 `multiset` 的 `insert` 和 `count` 函数时间复杂度较高。

```cpp
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        // nums1 + nums2 = - nums3 - nums4
        unordered_map<int, int> um;
        int res = 0;
        int nums_size = nums1.size();
        // 先遍历 nums1，nums2，保存所有可能取值
        for (int i = 0; i < nums_size; ++i)
        {
            for (int j = 0; j < nums_size; ++j)
            {
                ++um[nums1[i] + nums2[j]];
            }
        }
        // 后遍历 nums3，nums4，查找哈希集合中是否有其相反数
        for (int i = 0; i < nums_size; ++i)
        {
            for (int j = 0; j < nums_size; ++j)
            {
                int opp_count = um[0 - nums3[i] - nums4[j]];
                // 如果找到了，并且有可能不止一个
                if (opp_count)
                    res += opp_count;
            }
        }

        return res;
    }
};
```