---
title: 【刷题日记】二叉树-翻转二叉树-L226-Easy
tags:
  - C++
  - 刷题
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
comments: false
cover: false
date: 2024-10-30 10:26:03
---

给你一棵二叉树的根节点 root ，翻转这棵二叉树，并返回其根节点。

<!-- more -->

---

## 思路

* 肯定是用递归。刚开始考虑到左右子树存在 nullptr 的情况，不过发现这种情况也包含在正常情况中了，无需特殊考虑。

## 学习点



## 代码

```cpp
class Solution
{
public:
    TreeNode *invertTree(TreeNode *root)
    {
        // 递归终止条件
        if (!root)
            return root;
        // 递归子树
        invertTree(root->left);
        invertTree(root->right);
            
        TreeNode *tmp_node = root->left;
        root->left = root->right;
        root->right = tmp_node;

        return root;
    }
};
```