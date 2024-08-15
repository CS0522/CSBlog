---
title: 【刷题日记】字符串-重复的子字符串-L459-Easy
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
date: 2024-08-15 12:08:22
---

给定一个非空的字符串 s ，检查是否可以通过由它的一个子串重复多次构成。

<!-- more -->

---

## 思路

* 多次循环 `KMP` 算法，重点在于如何取子字符串作为模式串进行匹配，减少时间开销。

## 学习点

* 如何取子字符串？子字符串的长度一定是母串的因数，这样可以剪枝处理，节省很多循环。

## 代码

我的（复杂了）：

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
    bool KMP(string &text, string &pattern, int *next)
    {
        // int next[pattern.length()];
        // // 计算 next 数组
        // calNext(next, pattern);
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
                return true;
            }
        }
        // 匹配失败
        return false;
    }


    bool repeatedSubstringPattern(string s) {
        int len = s.length();
        bool flag = false;
        // i 为长度
        for (int i = len / 2; i >= 1; i--)
        {
            // NOTE: 剪枝处理
            // 只有倍数才满足要求
            // 可以节省很多循环
            if (len % i != 0)
                continue;
            string pattern = s.substr(0, i);
            // pattern 作为模式串
            // 用 KMP 算法进行匹配
            int next[pattern.length()];
            calNext(next, pattern);
            // end 为已经匹配成功的下标 + 1
            int end = i;
            while (end != len)
            {
                string text = s.substr(end, i);
                // 部分匹配失败
                // 退出 while，for 循环 continue
                if (!KMP(text, pattern, next))
                    break;
                // 部分匹配成功
                else
                    end += i;
            }
            // 匹配成功
            if (end == len)
                return true;
            // 该字串匹配失败
            else
                continue;
        }
        // 全部匹配失败
        return false;
    }
};
```

官解的 `KMP`：

```cpp
class Solution {
public:
    bool kmp(const string& query, const string& pattern) {
        int n = query.size();
        int m = pattern.size();
        vector<int> fail(m, -1);
        for (int i = 1; i < m; ++i) {
            int j = fail[i - 1];
            while (j != -1 && pattern[j + 1] != pattern[i]) {
                j = fail[j];
            }
            if (pattern[j + 1] == pattern[i]) {
                fail[i] = j + 1;
            }
        }
        int match = -1;
        for (int i = 1; i < n - 1; ++i) {
            while (match != -1 && pattern[match + 1] != query[i]) {
                match = fail[match];
            }
            if (pattern[match + 1] == query[i]) {
                ++match;
                if (match == m - 1) {
                    return true;
                }
            }
        }
        return false;
    }

    bool repeatedSubstringPattern(string s) {
        return kmp(s + s, s);
    }
};
```