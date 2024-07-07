---
title: 【刷题日记】哈希表-两数之和-L1-Easy
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
date: 2024-07-07 11:33:26
---

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

<!-- more -->

---

## 思路

* 时间复杂度较高的原因是寻找 `target - x` 的时间复杂度过高。因此，我们需要一种更优秀的方法，能够快速寻找数组中是否存在目标元素。如果存在，我们需要找出它的索引。创建一个哈希表，对于每一个 x，我们首先查询哈希表中是否存在 `target - x`，然后将 x 插入到哈希表中，即可保证不会让 x 和自己匹配。

## 学习点

* 使用哈希表存储已经访问过的元素，然后在哈希表中查找 `target - x`。

## 代码

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hashtable;
        for (int i = 0; i < nums.size(); ++i) {
            auto it = hashtable.find(target - nums[i]);
            if (it != hashtable.end()) {
                return {it->second, i};
            }
            hashtable[nums[i]] = i;
        }
        return {};
    }
};
```