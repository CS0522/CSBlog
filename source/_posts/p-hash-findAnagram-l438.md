---
title: 【刷题日记】哈希表-找到字符串中所有字母异位词-L438-Medium
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
date: 2024-07-06 21:23:00
---

给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

<!-- more -->

---

## 思路

* 与 L49 一致，但超出时间限制。

* 滑动窗口，固定一段长度，在前进的时候只要左边的字母数量自减，右边的字母数量自加即可，与刚开始的思路优化的是减少了每次都需要进行 key 的排序。实际上也是对哈希表的键的取值的优化。

## 学习点

* 构建哈希表的键的取值，滑动窗口在移动的同时只需要两端的字母的数量自减或自加即可，大大节省时间开销。

## 代码

超出时间限制：

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        if (s.length() < p.length())
          {
              return {};
          }
        int len = p.length();
        // unordered_map
        // key(string). value(index)
        unordered_map<string, vector<int>> um;
        // 遍历 s
        for (int i = 0; i < s.length(); i++)
        {
            string key = s.substr(i, len);
            sort(key.begin(), key.end());
            um[key].emplace_back(i);
        }
        // 找 p 对应的键值
        sort(p.begin(), p.end());
        return um[p];
    }
};
```

滑动窗口：

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        int s_len = s.length();
        int p_len = p.length();
        if (s_len < p_len)
            return {};
        // 滑动窗口 + 哈希表
        vector<int> s_counts(26, 0);
        vector<int> p_counts(26, 0);
        // 存储 indices
        vector<int> ans;
        // 初始化
        for (int i = 0; i < p_len; ++i)
        {
            ++s_counts[s[i] - 'a'];
            ++p_counts[p[i] - 'a'];
        }
        // 滑动窗口遍历 s
        for (int i = 0; i < s_len - p_len; ++i)
        {
            // 如果哈希数组相同
            if (s_counts == p_counts)
                ans.emplace_back(i);
            // 不相同，滑动窗口前进，前面的自减，后面的自加
            --s_counts[s[i] - 'a'];
            ++s_counts[s[i + p_len] - 'a'];
        }

        // 最后再比较一次，如果哈希数组相同
        if (s_counts == p_counts)
            ans.emplace_back(s_len - p_len);

        return ans;
    }
};
```