---
title: 【刷题日记】链表-链表相交-L160-Easy
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
date: 2024-06-29 12:07:14
---

给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。

<!-- more -->

---

[面试题 02.07. 链表相交](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/description/)

## 思路

* 用两个栈，从后往前的思路，分别存储指向 A，B 链表的指针。因为链表相交，则从相交节点往后的节点都是一致的，因此找到第一个不一致的节点，就是相交节点的前驱；如果其中某个栈空了都没有找到，则返回栈空的链表的指针，该指针指向的即为相交节点

* 哈希集合，用哈希集合存储链表节点。首先遍历链表 headA，并将链表 headA 中的每个节点加入哈希集合中。然后遍历链表 headB，对于遍历到的每个节点，判断该节点是否在哈希集合中。如果链表 headB 中的所有节点都不在哈希集合中，则两个链表不相交，返回 nullptr

## 学习点

* `unordered_set` 无序集合，构造哈希集合

## 代码

用栈，从后往前

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        // 用栈分别保存 A，B 链表的指针，从后往前对比
        stack<ListNode*> stA;
        stack<ListNode*> stB;

        ListNode* pA = headA;
        ListNode* pB = headB;

        while (pA != nullptr)
        {
            stA.push(pA);
            pA = pA->next;
        }
        while (pB != nullptr)
        {
            stB.push(pB);
            pB = pB->next;
        }
        // 弹出栈
        bool found = false;
        while (!stA.empty() && !stB.empty())
        {
            pA = stA.top();
            stA.pop();
            pB = stB.top();
            stB.pop();
            // 相交点的前驱的条件
            if ((pA != pB) && (pA->next == pB->next))
            {
                found = true;
                break;
            }
        }
        // 如果找到了，相交节点为 next
        if (found)
        {
            return (pA->next);
        }
        // 如果没找到，则返回当前指向节点
        else
        {
            if (stA.empty())
                return pA;
            else
                return pB;
        }
    }
};
```

无序集合，建立哈希集合

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        // 哈希集合
        unordered_set<ListNode*> uset;
        // 初始化哈希集合
        ListNode *p = headA;
        while (p != nullptr)
        {
            uset.insert(p);
            p = p->next;
        }
        // 遍历 headB
        p = headB;
        while (p != nullptr)
        {
            if (uset.count(p))
            {
                // 存在指向该节点的指针
                return p;
            }
            p = p->next;
        }
        return nullptr;
    }
};
```