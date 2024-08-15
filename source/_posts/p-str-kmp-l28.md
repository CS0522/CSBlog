---
title: 【刷题日记】字符串-找出字符串中第一个匹配项的下标-L28-Easy
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
date: 2024-08-14 18:12:43
---

给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回 -1 。

<!-- more -->

---

## 思路

* `KMP` 算法。

## 学习点

* `KMP` 算法。可以背模板。

## 代码

KMP：

```cpp
class Solution {
public:
    // 计算 next 数组
    void calNext(int *next, string &pattern)
    {
        // 回溯指针
        // 也可以表示最长前缀长度
        int j = -1;
        // 默认 next[0] = -1
        next[0] = -1;
        for (int i = 1; i < pattern.length(); i++)
        {
            // 当正在匹配的字符 p[i] 
            // 与正在匹配的最长前缀的下一位 p[j + 1] 
            // 匹配失败时，
            // j 回溯到最长前缀的上一个最长前缀
            while (j != -1 && pattern[i] != pattern[j + 1])
            {
                j = next[j];
            }
            // 匹配成功
            // 最长前缀 + 1
            if (pattern[i] == pattern[j + 1])
            {
                j += 1;
            }
            next[i] = j;
        }
    }

    // KMP
    // 返回匹配成功的下标
    int KMP(string &text, string &pattern)
    {
        int next[pattern.length()];
        // 计算 next 数组
        calNext(next, pattern);
        int j = -1;
        // 开始匹配
        for (int i = 0; i < text.length(); i++)
        {
            while (j != -1 && text[i] != pattern[j + 1])
            {
                j = next[j];
            }
            if (text[i] == pattern[j + 1])
            {
                j++;
            }
            // 当 j 匹配到模式串最后一位
            // 匹配成功
            if (j == pattern.length() - 1)
            {
                return i - j;
            }
        }
        // 匹配失败
        return -1;
    }

    int strStr(string haystack, string needle) {
        // KMP
        return KMP(haystack, needle);
    }
};
```