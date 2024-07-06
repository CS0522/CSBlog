---
title: 【刷题日记】哈希表-字母异位词分组-L49-Medium
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
date: 2024-07-06 10:25:18
---

给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

字母异位词 是由重新排列源单词的所有字母得到的一个新单词。

<!-- more -->

---

## 思路

* 构建哈希表 map

## 学习点

* 如何**构建哈希的 key 值**，来代表每组的 anagram。因为异位词都是字母相同，字母位置不同，所以按照**字典排序的结果**是一样的，可以当作键。

## 代码

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // 哈希表 map，key 为排序后的 str，值为对应的 vector<string>
        unordered_map<string, vector<string>> um;
        for (string str : strs)
        {
            string key = str;
            sort(key.begin(), key.end());
            um[key].emplace_back(str);
        }
        vector<vector<string>> res;
        for (auto it = um.begin(); it != um.end(); it++)
        {
            res.emplace_back(it->second);
        }
        return res;
    }
};
```