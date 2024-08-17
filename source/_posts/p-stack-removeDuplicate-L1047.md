---
title: 【刷题日记】栈和队列-删除字符串中的所有相邻重复项-L1047-Easy
tags:
  - C++
  - 刷题
  - 栈和队列
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 栈和队列
comments: false
cover: false
date: 2024-08-17 00:50:19
---

给出由小写字母组成的字符串 S，重复项删除操作会选择两个相邻且相同的字母，并删除它们。

在 S 上反复执行重复项删除操作，直到无法继续删除。

在完成所有重复项删除操作后返回最终的字符串。答案保证唯一。

<!-- more -->

---

## 思路

* 用栈存放遍历过的元素，当遍历当前的这个元素的时候，去栈中检查是否遍历过相同数值的相邻元素，再做消除操作。

## 学习点

* 栈

## 代码

```cpp
class Solution {
public:
    string removeDuplicates(string s) {
        // 用栈
        stack<char> uniques;
        for (int i = 0; i < s.length(); i++)
        {
            if ((!uniques.empty() && s[i] != uniques.top()) || uniques.empty())
            {
                uniques.push(s[i]);
            }
            else if (!uniques.empty() && s[i] == uniques.top())
            {
                uniques.pop();
            }
        }
        // stack 中即为最后的结果
        string res = "";
        while (!uniques.empty())
        {
            res += uniques.top();
            uniques.pop();
        }
        reverse(res.begin(), res.end());

        return res;
    }
};
```