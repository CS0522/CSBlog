---
title: 【刷题日记】哈希表-有效的字母异位词-L242-Easy
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
date: 2024-07-06 10:11:46
---

给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

<!-- more -->

---

## 思路

* 建立哈希表，保存每个字符出现的次数。s 中的字母出现则增加，t 中的字母出现则减少。如果最后哈希表中刚好都为 0，则满足条件。

## 学习点



## 代码

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.length() != t.length())
        {
            return false;
        }
        // 建立哈希表，保存每个字符出现的次数
        int char_count[26] = {0};
        for (int i = 0; i < s.length(); ++i)
        {
            char_count[s[i] - 'a'] += 1;
        }
        for (int i = 0; i < t.length(); ++i)
        {
            char_count[t[i] - 'a'] -= 1;
        }
        // 如果刚好所有 char_count 里的值均为 0
        // 则 s，t 为异位词
        for (int i = 0; i < 26; ++i)
        {
            if (char_count[i])
            {
                return false;
            }
        }
        return true;
    }   
};
```