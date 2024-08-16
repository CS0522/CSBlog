---
title: 【刷题日记】栈和队列-有效括号-L20-Easy
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
date: 2024-08-16 18:27:53
---

给定一个只包括 `'('，')'，'{'，'}'，'['，']'` 的字符串 s ，判断字符串是否有效。

<!-- more -->

---

## 思路

* 用栈暂存左括号；用 map 键值对存储左括号和右括号的对应关系。

## 学习点

* 栈、map。

## 代码

```cpp
class Solution {
public:
    bool isValid(string s) {
        // 用类似 map 的数据结构
        // 保存对应关系
        map<char, char> brace_mp;
        brace_mp.insert(make_pair<>('(', ')'));
        brace_mp.insert(make_pair<>('[', ']'));
        brace_mp.insert(make_pair<>('{', '}'));

        stack<char> left_brace_st;
        for (int i = 0; i < s.length(); i++)
        {
            // 左括号入栈
            // 右括号弹出栈
            if (brace_mp.count(s[i]))   // 左括号
                left_brace_st.push(s[i]);
            
            else                        // 右括号
            {
                if (!left_brace_st.empty())
                {
                    auto tmp_left_brace = left_brace_st.top();

                    if (s[i] != brace_mp[tmp_left_brace])
                        break;          // 跳出循环，残留元素，统一 return false
                    else
                        left_brace_st.pop();
                }
                else
                    return false;
            }
        }

        return left_brace_st.empty();
    }
};
```