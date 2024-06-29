---
title: 【刷题日记】链表-删除链表的倒数第N个节点-L19-Medium
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
date: 2024-06-28 21:25:37
---

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

<!-- more -->

---

[19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/)

## 思路

* 两次遍历，第 1 次计算总数目，第 2 次删除指定节点
* 用栈倒序存储指针，栈内正数第 N 个指针即指向删除的目标节点；弹出指针，最后返回头节点

**update**
* 双指针。快指针始终比慢指针快 n 个节点。这样只需要遍历一遍

## 学习点

* 栈存储指针
* 倒数 n 个节点，n 这个值保持不变，双指针节省遍历次数

## 代码

两次遍历

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // 两遍遍历
        // 1 取得个数
        ListNode* p = head;
        int count = 0;
        while (p != nullptr)
        {
            count += 1;
            p = p->next;
        }
        // 2 删除节点
        // 创建头节点
        ListNode* head_node = new ListNode(-1, head);
        p = head_node;
        int target_index = count - n;
        while (target_index > 0)
        {
            p = p->next;
            target_index -= 1;
        }
        // 删除他的下一个节点
        ListNode* target = p->next;
        p->next = target->next;
        delete target;

        return head_node->next;
    }
};
```

栈

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // 栈，存储指针
        stack<ListNode*> st;
        // 添加头节点
        ListNode* head_node = new ListNode(-1, head);
        ListNode* p = head_node;
        
        while (p != nullptr)
        {
            st.push(p);
            p = p->next;
        }
        // 弹出栈
        for (int i = 1; i <= n; i++)
        {
            st.pop();
        }
        // 待删除节点的前驱
        ListNode* pre = st.top();
        ListNode* target = pre->next;
        pre->next = target->next;
        delete target;

        return head_node->next;
    }
};
```

双指针

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // 双指针
        // 添加头节点
        ListNode* head_node = new ListNode(-1, head);
        // 初始化快慢指针
        ListNode* slow = head_node;
        ListNode* fast = slow;
        while (n >= 0)
        {
            fast = fast->next;
            --n;
        }
        // 遍历
        while (fast != nullptr)
        {
            // forward
            fast = fast->next;
            slow = slow->next;
        }
        // fast 指向表尾 nullptr
        // slow 指向待删除元素前驱
        ListNode* target = slow->next;
        slow->next = target->next;
        delete target;

        return head_node->next;
    }
};
```