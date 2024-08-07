---
title: 【刷题日记】链表-反转链表-L206-Easy
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
date: 2024-06-25 01:05:00
---

给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。

<!-- more -->

---

## 思路

* 交换值。栈，遍历 2 遍，时间复杂度 `O(n)`，空间复杂度 `O(n)`
* 反转指针

## 学习点

* `ans = new ListNode(x.val, ans)`，当前指向节点A（或 null），构建新节点B，其 next 指针指向节点A（或 null），同时指针指向新节点B

## 代码

栈：

```cpp
// 栈
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        // 栈
        // 交换 val
        stack<int> s;
        // 遍历 2 遍
        ListNode *p = head;
        while (p != nullptr)
        {
            s.push(p->val);
            p = p->next;
        }
        ListNode *q = head;
        while (q != nullptr && !s.empty())
        {
            q->val = s.top();
            s.pop();
            q = q->next;
        }

        return head;
    }
};
```

评论区更简单解法：

```cpp
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode ans = nullptr;
        for (ListNode x = head; x != nullptr; x = x.next) {
            // 新建 node 的 next 域指向当前 node
            // ans 指向新建 node
            // 以此达到反转指针的目的
            ans = new ListNode(x.val, ans);
        }
        return ans;
    }
}
```

反转指针：

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        // pre 指向前驱点
        ListNode *pre = nullptr;
        ListNode *p = head;

        while (p != nullptr)
        {
            // 暂存下一个节点的指针
            ListNode *p_next = p->next;
            // 反转指针
            p->next = pre;
            // 指针移动
            pre = p;
            p = p_next;
        }

        return pre;
    }
};
```