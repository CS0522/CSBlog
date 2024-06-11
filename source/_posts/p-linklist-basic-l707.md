---
title: 【刷题日记】链表-设计链表-L707-Medium
tags:
  - C++
  - 刷题
  - 链表
  - 链表基础
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 链表
  - 链表基础
comments: false
cover: false
date: 2024-06-11 17:04:25
---

你可以选择使用单链表或者双链表，设计并实现自己的链表。

单链表中的节点应该具备两个属性：val 和 next 。val 是当前节点的值，next 是指向下一个节点的指针/引用。

如果是双向链表，则还需要属性 prev 以指示链表中的上一个节点。假设链表中的所有节点下标从 0 开始。

<!-- more -->

---

[707. 设计链表](https://leetcode.cn/problems/design-linked-list/description/)

## 思路

* 构造含有头节点的双链表

## 学习点

* 链表的基本操作

## 代码

```cpp
struct DNode
{
    int val;
    DNode* next;
    DNode* prev;

    DNode(int _val): val(_val) {}
    DNode(int _val, DNode* _next, DNode* _prev): 
                val(_val), next(_next), prev(_prev) {}
};

// 带头节点
class MyLinkedList {
public:
    MyLinkedList() {
        // 初始化一个空节点
        this->head = new DNode(-1, nullptr, nullptr);
        this->size = 0;
    }
    
    int get(int index) {
        if (index < 0 || index >= this->size ||
            this->head->next == nullptr)
        {
            return -1;
        }
        auto p = this->head;
        while (index >= 0 && p != nullptr)
        {
            p = p->next;
            --index;
        }
        return p->val;
    }
    
    void addAtHead(int val) {
        DNode* new_node = new DNode(val, nullptr, nullptr);
        // 设置前驱后继
        new_node->next = this->head->next;
        new_node->prev = this->head;
        // 修改链表前驱后继
        if (this->head->next != nullptr)
        {
            this->head->next->prev = new_node;
        }
        this->head->next = new_node;
        // 增加个数
        ++this->size;
    }
    
    void addAtTail(int val) {
        int count = this->size;
        auto p = this->head;
        // 移动到末尾元素
        while (count > 0)
        {
            p = p->next;
            --count;
        }
        // 添加元素
        DNode* new_node = new DNode(val, nullptr, nullptr);
        // 设置前驱
        new_node->prev = p;
        // 链接节点
        p->next = new_node;
        // 增加个数
        ++this->size;
    }
    
    void addAtIndex(int index, int val) {
        if (index > this->size || index < 0)
        {
            return;
        }
        if (index == this->size)
        {
            addAtTail(val);
            return;
        }
        auto p = head;
        // 遍历到 index - 1 处的元素
        while (index > 0)
        {
            p = p->next;
            --index;
        }
        // p 指向 index - 1 处的元素
        DNode* new_node = new DNode(val, nullptr, nullptr);
        p->next->prev = new_node;
        new_node->next = p->next;
        new_node->prev = p;
        p->next = new_node;
        // 增加个数
        ++this->size;
    }
    
    void deleteAtIndex(int index) {
        if (index < 0 || index >= this->size)
        {
            return;
        }
        auto p = head;
        while (index >= 0)
        {
            p = p->next;
            --index;
        }
        // p 指向要删除的节点
        // 前后节点链接
        p->prev->next = p->next;
        if (p->next != nullptr)
        {
            p->next->prev = p->prev;
        }
        // 释放 p
        delete p;
        // 减少个数
        --this->size;
    }

private:
    DNode* head;
    int size;
};

/**
 * Your MyLinkedList object will be instantiated and called as such:
 * MyLinkedList* obj = new MyLinkedList();
 * int param_1 = obj->get(index);
 * obj->addAtHead(val);
 * obj->addAtTail(val);
 * obj->addAtIndex(index,val);
 * obj->deleteAtIndex(index);
 */
```