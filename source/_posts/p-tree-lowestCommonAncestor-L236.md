---
title: 【刷题日记】二叉树-二叉树的最近公共祖先-L236-Medium
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
date: 2025-10-23 09:02:40
---

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

<!-- more -->

---

## 思路

* 先找到 p、q 节点，通过函数返回值返回；然后如果某个节点的左右递归函数均有返回值，则该节点为最近公共祖先，层层返回即可

## 学习点



## 代码

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == NULL || root == p || root == q)
            return root;
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);

        if (left)
            // 此时左右都有结果，那么root即为最近公共祖先
            if (right)
                return root;
            else
                return left;
        else
            return right;
        return NULL;
    }
};
```