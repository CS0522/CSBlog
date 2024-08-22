---
title: 【刷题日记】二叉树-填充每个节点的下一个右侧节点指针-L116-Medium
tags:
  - C++
  - 刷题
  - 二叉树
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 二叉树
comments: false
cover: false
date: 2024-08-22 19:29:45
---

给定一个 完美二叉树 ，其所有叶子节点都在同一层，每个父节点都有两个子节点。填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。

初始状态下，所有 next 指针都被设置为 NULL。

<!-- more -->

---

## 思路

* 层序遍历，遍历过程中每层的每个节点的 next 指针指向队列中的头节点（他的同层右节点）；每层的最后一个节点需要特殊处理。

## 学习点



## 代码

```cpp
// Definition for a Node.
class Node {
public:
    int val;
    Node* left;
    Node* right;
    Node* next;

    Node() : val(0), left(NULL), right(NULL), next(NULL) {}

    Node(int _val) : val(_val), left(NULL), right(NULL), next(NULL) {}

    Node(int _val, Node* _left, Node* _right, Node* _next)
        : val(_val), left(_left), right(_right), next(_next) {}
};


class Solution {
public:
    Node* connect(Node* root) {
        queue<Node*> qe;
        if (root)
            qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                // 设置他的 next 指针
                if (!qe.empty() && i != node_num - 1)
                {
                    auto tmp_next = qe.front();
                    tmp->next = tmp_next;
                }
                
                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }
        }

        return root;
    }
};
```