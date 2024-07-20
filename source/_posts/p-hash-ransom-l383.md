---
title: 【刷题日记】哈希表-赎金信-L383-Easy
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
date: 2024-07-08 10:58:43
---

给你两个字符串：ransomNote 和 magazine ，判断 ransomNote 能不能由 magazine 里面的字符构成。

<!-- more -->

---

## 思路

* 建立哈希表存储 ransomNote 中的字母以及出现的个数；遍历 magazine，当出现相同字母的时候，自减；当哈希表中所有键的值都小于等于 0 时，说明可以满足条件。

* 只需要满足字符串 magazine 中的每个英文字母 (’a’-’z’) 的统计次数都大于等于 ransomNote 中相同字母的统计次数即可。

## 学习点

* 反向思路

## 代码

初始思路，时间复杂度较高，主要来自于 um_count_leq0 函数。

```cpp
class Solution {
public:
    int um_count_leq0(unordered_map<char, int> &um)
    {
        int count = 0;
        for (auto it = um.begin(); it != um.end(); it++)
        {
            if (it->second <= 0)
                ++count;
        }
        return count;   
    }

    bool canConstruct(string ransomNote, string magazine) {
        // 用哈希表 unordered_map 记录字母出现次数
        unordered_map<char, int> um;
        // 遍历 ransomNote，添加哈希键值对
        for (int i = 0; i < ransomNote.length(); ++i)
        {
            ++um[ransomNote[i]];
        }
        // 遍历 magazine，当出现哈希集合中全部为 0 的时候，返回 true
        for (int i = 0; i < magazine.length(); ++i)
        {
            // 当哈希集合中存在该字母
            if (um.count(magazine[i]))
                --um[magazine[i]];
            if (um_count_leq0(um) == um.size())
                return true;
        }
        return false;
    }
};
```

题解思路：

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        if (ransomNote.length() > magazine.length())
            return false;
        vector<int> counts(26, 0);
        for (int i = 0; i < magazine.length(); ++i)
        {
            ++counts[magazine[i] - 'a'];
        }
        for (int i = 0; i < ransomNote.length(); ++i)
        {
            --counts[ransomNote[i] - 'a'];
            // 如果 < 0，说明 magazine 中的字母数不够
            if (counts[ransomNote[i] - 'a'] < 0)
                return false;
        }
        return true;
    }
};
```