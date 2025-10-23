---
title: 【刷题日记】二叉树-二叉搜索树的最小绝对差-L530-Easy
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
date: 2025-10-23 09:00:27
---

给你一个二叉搜索树的根节点 root ，返回 树中任意两不同节点值之间的最小差值。

差值是一个正数，其数值等于两值之差的绝对值。

<!-- more -->

---

## 思路

* 中序遍历，转换为递增的数组，然后比较两个元素之间的最小差值

## 学习点



## 代码

```cpp
class Solution {
public:
    // 中序遍历顺序
    std::vector<int> tree_nodes;
    void traverseBST(TreeNode *root) {
        if (root == NULL)
            return;
        
        traverseBST(root->left);
        tree_nodes.push_back(root->val);
        traverseBST(root->right);
    }
    int getMinimumDifference(TreeNode* root) {
        int min_diff = INT_MAX;
        traverseBST(root);
        for (size_t i = tree_nodes.size() - 1; i >= 1; i--) {
            if (std::abs(tree_nodes[i] - tree_nodes[i - 1]) < min_diff)
                min_diff = std::abs(tree_nodes[i] - tree_nodes[i - 1]);
        }
        return min_diff;
    }
};
```