---
title: 【刷题日记】链表-两两交换链表中的节点-L24-Medium
tags:
  - C++
  - 刷题
  - 链表
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 链表
comments: false
cover: false
date: 2024-06-25 10:58:30
---

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

<!-- more -->

---

[24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/description/)

## 思路

* 修改指针指向

## 学习点

* 添加一个空的头节点，方便统一交换，不需要单独针对 head 节点设置

## 代码

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        // 新建头节点
        ListNode *head_node = new ListNode(-1, head);
        // 前一个节点
        ListNode *pre = head_node;
        // 当前节点
        ListNode *curr = head;

        while (curr != nullptr)
        {
            // 下一个节点
            ListNode *post = curr->next;
            // post 不为 null
            if (post != nullptr)
            {
                curr->next = post->next;
                post->next = curr;
                pre->next = post;
            }
            // forward
            pre = curr;
            curr = curr->next;
        }

        return head_node->next;
    }
};
```