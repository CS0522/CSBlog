---
title: 【刷题日记】栈和队列-用队列实现栈-L225-Easy
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
date: 2024-08-16 16:35:34
---

请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（push、top、pop 和 empty）。

<!-- more -->

---

## 思路

* 用一个队列，将队头元素弹出并重新加入队尾，并用 `count` 计数，当队头元素为最后一个元素时，弹出。

## 学习点



## 代码

```cpp
class MyStack {
private:
    queue<int> inQ;

public:
    MyStack() {
    }
    
    void push(int x) {
        inQ.push(x);
    }
    
    int pop() {
        auto inQ_size = inQ.size();
        // 返回最后一个元素
        int ele_cnt = 0;
        while (ele_cnt != inQ_size - 1)
        {
            inQ.push(inQ.front());
            inQ.pop();

            ele_cnt += 1;
        }
        // 此时队头元素为要 pop 的元素
        auto res = inQ.front();
        inQ.pop();

        return res;
    }
    
    int top() {
        auto inQ_size = inQ.size();
        // 返回最后一个元素
        int ele_cnt = 0;
        while (ele_cnt != inQ_size - 1)
        {
            inQ.push(inQ.front());
            inQ.pop();

            ele_cnt += 1;
        }
        // 此时队头元素为要 pop 的元素
        auto res = inQ.front();
        
        inQ.push(inQ.front());
        inQ.pop();

        return res;
    }
    
    bool empty() {
        return inQ.empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack* obj = new MyStack();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->top();
 * bool param_4 = obj->empty();
 */
```