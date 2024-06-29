---
title: 【刷题日记】链表-移除链表元素-L203-Easy
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
date: 2024-06-11 16:41:06
---

给你一个链表的头节点 head 和一个整数 val ，请你删除链表中所有满足 Node.val == val 的节点，并返回 新的头节点 。

<!-- more -->

---

## 思路



## 学习点



## 代码

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        // 处理下一个元素，这样避免特殊处理最后一个元素
        while (head != nullptr && head->val == val)
        {
            head = head->next;
        }
        if (head == nullptr)
        {
            return head;
        }

        // 此时 head 元素不为 val
        // pointer
        ListNode* p = head;
        while (p->next != nullptr)
        {
            // 移除元素
            if (p->next->val == val)
            {
                auto temp = p->next;
                // 链接下下个元素
                p->next = p->next->next;
                // 释放下个元素
                delete temp;
            }
            else
            {
                p = p->next;
            }
        }
        return head;
    }
};
```