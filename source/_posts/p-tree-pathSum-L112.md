---
title: 【刷题日记】二叉树-路径总和-L112-Easy
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
date: 2024-11-02 17:14:40
---

给你二叉树的根节点 root 和一个表示目标和的整数 targetSum 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 targetSum 。如果存在，返回 true ；否则，返回 false 。

<!-- more -->

---

## 思路

* 思路 1
    * 不需要记录每个路径的总和，只需要判断即可。因此在递归的时候，`targetSum - current_node->val`。路径结束的节点为叶子节点，而只有左孩子或右孩子的节点不是路径结束的节点，因此要注意这里的处理：只有左右节点都为空时，返回节点值和路径和的判断；若有一个不为空，则继续递归，其中节点为空的路径会返回 false。而在递归左右节点的时候，采用 `||`，只要有一个路径满足即可返回 `true`。

## 学习点



## 代码

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root)
            return false;
        // 必须是叶子节点
        if (!root->left && !root->right)
            return (root->val == targetSum);
        return (hasPathSum(root->left, targetSum - root->val) ||
                hasPathSum(root->right, targetSum - root->val));
    }
};
```