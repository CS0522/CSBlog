---
title: 【刷题日记】栈和队列-用栈实现队列-L232-Easy
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
date: 2024-08-16 16:33:03
---

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：

<!-- more -->

---

## 思路

* 两个栈，互相倒腾。

## 学习点

* 栈和队列的特性。

## 代码

```cpp
class MyQueue {
private:
    stack<int> st;
    stack<int> tmp_st;
    
public:
    MyQueue() {
    }
    
    void push(int x) {
        this->st.push(x);
    }
    
    int pop() {
        while (!st.empty())
        {
            tmp_st.push(st.top());
            st.pop();
        }
        auto res = tmp_st.top();
        tmp_st.pop();
        // 元素还回 st
        while (!tmp_st.empty())
        {
            st.push(tmp_st.top());
            tmp_st.pop();
        }

        return res;
    }
    
    int peek() {
        while (!st.empty())
        {
            tmp_st.push(st.top());
            st.pop();
        }
        auto res = tmp_st.top();
        // tmp_st.pop();
        // 元素还回 st
        while (!tmp_st.empty())
        {
            st.push(tmp_st.top());
            tmp_st.pop();
        }

        return res;
    } 
    
    bool empty() {
        return (st.empty() && tmp_st.empty());
    }
};

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue* obj = new MyQueue();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->peek();
 * bool param_4 = obj->empty();
 */
```