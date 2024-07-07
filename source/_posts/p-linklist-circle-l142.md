---
title: 【刷题日记】链表-环形链表II-L142-Medium
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
date: 2024-07-03 20:06:10
---

给定一个链表的头节点 head，返回链表开始入环的第一个节点。如果链表无环，则返回 null。

<!-- more -->

---

## 思路

* 哈希表，存储已经遍历过的 ListNode 指针。

* [快慢指针，追及问题。。。](https://programmercarl.com/0142.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8II.html#%E6%80%9D%E8%B7%AF)

## 学习点

* 快慢指针，快指针和慢指针最终都会相遇。

## 代码

哈希表

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        // 哈希集合保存已经出现过的指针
        ListNode *p = head;
        std::unordered_set<ListNode*> us;

        while (p != nullptr)
        {
            // 如果哈希集合中存在这个指针
            if (us.count(p))
                return p;
            // 不存在
            us.insert(p);
            p = p->next;
        }
        // 遍历结束，没有环
        return p;
    }
};
```

快慢指针

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        // 快慢指针
        ListNode* fast = head;
        ListNode* slow = head;
        while (fast != nullptr && fast->next != nullptr)
        {
            fast = fast->next->next;
            slow = slow->next;
            // 相遇
            if (slow == fast)
            {
                ListNode* index1 = head;
                ListNode* index2 = fast;
                while (index1 != index2)
                {
                    index1 = index1->next;
                    index2 = index2->next;
                }
                return index1;
            }
        } 
        return nullptr;
    }
};
```