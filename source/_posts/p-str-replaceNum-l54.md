---
title: 【刷题日记】字符串-替换数字-L54-Easy
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
date: 2024-08-14 17:58:01
---

给定一个字符串 s，它包含小写字母和数字字符，请编写一个函数，将字符串中的字母字符保持不变，而将每个数字字符替换为number。 例如，对于输入字符串 "a1b2c3"，函数应该将其转换为 "anumberbnumbercnumber"。

<!-- more -->

---

## 思路

* 原地修改字符串，不使用额外空间。

## 学习点

* 双指针，从后往前进行填充元素。

## 代码

```cpp
#include <iostream>
using namespace std;

int main()
{
    string s;
    while (cin >> s)
    {
        // 统计字符串中数字个数
        int nums_count = 0;
        int old_index = s.length() - 1;
        for (int i = 0; i < s.length(); i++)
        {
            if (s[i] >= '0' && s[i] <= '9')
                nums_count += 1;
        }
        // 扩容
        s.resize(s.length() - nums_count + 6 * nums_count);
        
        // 从后向前进行数字替换
        int new_index = s.length() - 1;
        while (old_index >= 0)
        {
            if (s[old_index] >= '0' && s[old_index] <= '9')
            {
                s[new_index--] = 'r';
                s[new_index--] = 'e';
                s[new_index--] = 'b';
                s[new_index--] = 'm';
                s[new_index--] = 'u';
                s[new_index--] = 'n';
            }
            else
                s[new_index--] = s[old_index];
            old_index -= 1;
        }
        cout << s << endl;
    }
    return 0;
}
```