---
title: 【刷题日记】二叉树-二叉搜索树中的搜索-L700-Easy
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
date: 2025-05-16 21:35:35
---

给定二叉搜索树（BST）的根节点 root 和一个整数值 val。

你需要在 BST 中找到节点值等于 val 的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 null 。

<!-- more -->

---

## 思路

迭代法

## 学习点



## 代码

```cpp
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        TreeNode* p = root;
        while (p)
        {
            if (p->val == val)
                return p;
            else if (val < p->val)
                p = p->left;
            else
                p = p->right;
        }
        return nullptr;
    }
};
```