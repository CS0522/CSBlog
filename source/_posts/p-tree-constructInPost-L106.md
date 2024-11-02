---
title: 【刷题日记】二叉树-从中序与后序遍历序列构造二叉树-L106-Medium
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
date: 2024-11-02 19:16:47
---

给定两个整数数组 inorder 和 postorder，其中 inorder 是二叉树的中序遍历，postorder 是同一棵树的后序遍历，请你构造并返回这颗 二叉树 。

<!-- more -->

---

## 思路

* 思路 1
    * 递归构建树，每层递归返回构建的根节点，递归中构建根节点的左右子节点。后序遍历中最后一个节点为当前递归层的根节点，在中序遍历中找到这个根节点（给定条件 树中没有重复元素），然后中序遍历中这个根节点的左边都为根节点的左子树，根节点的右边都为根节点的右子树，因此在准备递归构造左子树和右子树前，删除后序遍历的最后一个节点（为当前根节点，已经用过），将中序遍历分割为左右两部分，然后两部分分别作为参数、和后序遍历一起传入到下一层递归中。递归终止条件即为中序遍历已经空了（不判断后序遍历，因为左子树构造完但右子树可能没构造完）。

## 学习点

* 后序遍历最后一个节点为根节点；以此来分割中序遍历。

## 代码

思路 1：

```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        // 空节点
        if (!inorder.size())
            return nullptr;
        // 后序遍历的最后一个节点为当前的根节点
        int root_val = postorder[postorder.size() - 1];
        postorder.pop_back();
        TreeNode* root = new TreeNode(root_val);
        // 后序遍历顺序：左右根，反构造时先构造右节点
        vector<int>::iterator root_iter = find(inorder.begin(), inorder.end(), root_val);
        vector<int> left_inorder(inorder.begin(), root_iter);
        vector<int> right_inorder(root_iter + 1, inorder.end());
        
        root->right = buildTree(right_inorder, postorder);
        root->left = buildTree(left_inorder, postorder);

        return root;
    }
};
```

思路 1 优化版：

不需要每次新构建 vectors，直接传入分割的下标。

```cpp
class Solution {
public:
    TreeNode * build_tree(vector<int> &inorder, vector<int> &postorder, 
                        int inorder_start, int inorder_end)
    {
        // 空节点
        if ((inorder_end - inorder_start) == 0)
            return nullptr;
        // 后序遍历的最后一个节点为当前的根节点
        int root_val = postorder[postorder.size() - 1];
        postorder.pop_back();
        TreeNode *root = new TreeNode(root_val);
        // 分割中序遍历
        vector<int>::iterator root_iter = find(inorder.begin() + inorder_start, inorder.begin() + inorder_end, root_val);
        int root_index = (root_iter - inorder.begin());

        root->right = build_tree(inorder, postorder, root_index + 1, inorder_end);
        root->left = build_tree(inorder, postorder, inorder_start, root_index);

        return root;
    }

    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        return build_tree(inorder, postorder, 0, inorder.size());
    }
};
```