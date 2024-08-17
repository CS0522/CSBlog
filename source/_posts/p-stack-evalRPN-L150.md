---
title: 【刷题日记】栈和队列-逆波兰表达式求值-L150-Medium
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
date: 2024-08-17 16:28:02
---

给你一个字符串数组 tokens ，表示一个根据 逆波兰表示法 表示的算术表达式。

请你计算该表达式。返回一个表示表达式值的整数。

<!-- more -->

---

## 思路

* 用栈，保存数字；遇到运算符，弹出两个数运算后，结果入栈。结束后栈里的最后一个元素为结果。

## 学习点

* 可以用 `std::stoi()、std::stoll()` 转换为数字，不需要自己再写。

## 代码

```cpp
#include <iostream>
#include <vector>
#include <cstring>
#include <algorithm>
#include <stack>
#include <map>
using namespace std;

class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int> operands;
        // 存储运算符映射关系
        map<string, int> operator_map;
        operator_map.insert(make_pair<>("+", 1));
        operator_map.insert(make_pair<>("-", 2));
        operator_map.insert(make_pair<>("*", 3));
        operator_map.insert(make_pair<>("/", 4));

        for (int i = 0; i < tokens.size(); i++)
        {
            // 非运算符入栈
            if (!operator_map.count(tokens[i]))
                operands.push(stoi(tokens[i]));
            // 操作符弹出栈运算后入栈
            else
            {
                auto op1 = operands.top();
                operands.pop();
                auto op2 = operands.top();
                operands.pop();
                int res;
                
                switch (operator_map[tokens[i]])
                {
                case 1: // +
                    res = op2 + op1;
                    operands.push(res);
                    break;
                case 2: // -
                    res = op2 - op1;
                    operands.push(res);
                    break;
                case 3: // *
                    res = op2 * op1;
                    operands.push(res);
                    break;
                case 4: // /
                    res = op2 / op1;
                    operands.push(res);
                    break;
                }
            }
        }

        return operands.top();
    }
};
```