---
title: 【刷题日记】字符串-反转字符串-L344-Easy
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
date: 2024-08-14 17:56:33
---

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 s 的形式给出。

不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。

<!-- more -->

---

## 思路

* 临时变量
* 位运算，异或

## 学习点



## 代码

位运算：

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        int str_len = s.size();
        char tmp;
        for (int i = 0; i < str_len / 2; i++)
        {
            s[i] ^= s[str_len - 1 - i];
            s[str_len - 1 - i] ^= s[i];
            s[i] ^= s[str_len - 1 -i];
        }
    }
};
```

临时变量：

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        int str_len = s.size();
        char tmp;
        for (int i = 0; i < str_len / 2; i++)
        {
            tmp = s[i];
            s[i] = s[str_len - 1 -i];
            s[str_len - 1 -i] = tmp;
        }
    }
};
```