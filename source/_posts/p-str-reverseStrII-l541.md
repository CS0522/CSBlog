---
title: 【刷题日记】字符串-反转字符串II-L541-Easy
tags:
  - C++
  - 刷题
  - 字符串
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 字符串
comments: false
cover: false
date: 2024-08-14 17:57:34
---

给定一个字符串 s 和一个整数 k，从字符串开头算起，每计数至 2k 个字符，就反转这 2k 字符中的前 k 个字符。

如果剩余字符少于 k 个，则将剩余字符全部反转。
如果剩余字符小于 2k 但大于或等于 k 个，则反转前 k 个字符，其余字符保持原样。

<!-- more -->

---

## 思路

* 按照题目思路进行模拟

## 学习点



## 代码

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {
        int n = s.length();
        for (int i = 0; i < n; i += 2 * k) {
            reverse(s.begin() + i, s.begin() + min(i + k, n));
        }
        return s;
    }
};
```