---
title: 【刷题日记】哈希表-两个数组的交集-L349-Easy
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
date: 2024-07-06 23:23:17
---

给定两个数组 nums1 和 nums2 ，返回它们的交集。输出结果中的每个元素一定是 唯一 的。我们可以 不考虑输出结果的顺序 。

<!-- more -->

---

## 思路

* 使用哈希集合，分别遍历一次数组1和数组2，第一次遍历初始化哈希集合，第二次遍历对比交集。再使用另一个集合确保是还没有加入过结果中的元素，避免重复。时间复杂度为 `O(m + n)`。

## 学习点

* `vector<int>(us_no_repeat.begin(), us_no_repeat.end())` vector 利用迭代器的构造方法。

## 代码

哈希集合：

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        // 哈希集合
        unordered_set<int> us;
        // 为了去除重复
        // 结果集合
        unordered_set<int> us_no_repeat;
        // 遍历数组1，加入到哈希集合
        for (int i = 0; i < nums1.size(); i++)
        {
            us.insert(nums1[i]);
        }
        // 遍历数组2，查找相同元素
        for (int i = 0; i < nums2.size(); i++)
        {
            if (us.count(nums2[i]) && !us_no_repeat.count(nums2[i]))
            {
                us_no_repeat.insert(nums2[i]);
            }
        }

        return vector<int>(us_no_repeat.begin(), us_no_repeat.end());
    }
};
```