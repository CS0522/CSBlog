---
title: 【刷题日记】栈和队列-滑动窗口最大值-L239-Hard
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
date: 2024-08-17 20:13:07
---

给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

返回 滑动窗口中的最大值。

<!-- more -->

---

## 思路

* 用队列模拟滑动窗口，每添加一个元素就计算区间最大值。（超过时间限制）
* 使用 `deque` 实现 `单调队列（单调递减或单调递增的队列）`，放进去窗口里的元素，然后随着窗口的移动，队列也一进一出，每次移动之后，队列可以返回里面的最大值；每次窗口移动的时候，调用que.pop(滑动窗口中移除元素的数值)，que.push(滑动窗口添加元素的数值)，然后que.front()就返回最大值。**保证队列里的元素数值是由大到小的**。

## 学习点

* `单调队列`，队列中的元素从递增或者递减的顺序，有点类似于大顶堆、小顶堆，需要自己维护这个数据结构。

## 代码

超过时间限制：

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    int get_que_max(queue<int> &slide_que)
    {
        int ele_num = slide_que.size();
        int max_ele = slide_que.front();
        for (int i = 0; i < ele_num; i++)
        {
            auto ele_tmp = slide_que.front();
            slide_que.pop();
            if (max_ele < ele_tmp)
                max_ele = ele_tmp;
            slide_que.push(ele_tmp);
        }
        return max_ele;
    }

    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        queue<int> slide_que;
        vector<int> res;

        for (int i = 0; i < k; i++)
            slide_que.push(nums[i]);
        res.push_back(get_que_max(slide_que));
        for (int i = k; i < nums.size(); i++)
        {
            slide_que.pop();
            slide_que.push(nums[i]);
            res.push_back(get_que_max(slide_que));
        }

        return res;
    }
};
```

实现单调队列：

```cpp
class Solution {
private:
    class MyQueue { //单调队列（从大到小）
    public:
        deque<int> que; // 使用deque来实现单调队列
        // 每次弹出的时候，比较当前要弹出的数值是否等于队列出口元素的数值，如果相等则弹出。
        // 同时pop之前判断队列当前是否为空。
        void pop(int value) {
            if (!que.empty() && value == que.front()) {
                que.pop_front();
            }
        }
        // 如果push的数值大于入口元素的数值，那么就将队列后端的数值弹出，直到push的数值小于等于队列入口元素的数值为止。
        // 这样就保持了队列里的数值是单调从大到小的了。
        void push(int value) {
            while (!que.empty() && value > que.back()) {
                que.pop_back();
            }
            que.push_back(value);

        }
        // 查询当前队列里的最大值 直接返回队列前端也就是front就可以了。
        int front() {
            return que.front();
        }
    };
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        MyQueue que;
        vector<int> result;
        for (int i = 0; i < k; i++) { // 先将前k的元素放进队列
            que.push(nums[i]);
        }
        result.push_back(que.front()); // result 记录前k的元素的最大值
        for (int i = k; i < nums.size(); i++) {
            que.pop(nums[i - k]); // 滑动窗口移除最前面元素
            que.push(nums[i]); // 滑动窗口前加入最后面的元素
            result.push_back(que.front()); // 记录对应的最大值
        }
        return result;
    }
};
```