---
title: 【刷题日记】二叉树-从前序与中序遍历序列构造二叉树-L105-Medium
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
date: 2024-11-02 20:10:44
---

给定两个整数数组 preorder 和 inorder，其中 preorder 是二叉树的先序遍历，inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

<!-- more -->

---

## 思路

* 思路 1
    * 同 L106。

## 学习点

* 前序遍历第一个节点为根节点；以此来分割中序遍历。

## 代码

思路 1：

```cpp
class Solution {
public:
    TreeNode* build_tree(vector<int>& preorder, vector<int>& inorder, 
                        int inorder_start, int inorder_end)
    {
        if ((inorder_end - inorder_start) == 0)
            return nullptr;
        // 前序遍历的最后一个节点为当前的根节点
        int root_val = preorder[0];
        preorder.erase(preorder.begin());
        TreeNode* root = new TreeNode(root_val);
        // 前序遍历顺序：根左右，反构造时先构造左节点
        // 分割中序遍历
        vector<int>::iterator root_iter = find(inorder.begin() + inorder_start, inorder.begin() + inorder_end, root_val);
        int root_index = root_iter - inorder.begin();

        root->left = build_tree(preorder, inorder, inorder_start, root_index);
        root->right = build_tree(preorder, inorder, root_index + 1, inorder_end);

        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return build_tree(preorder, inorder, 0, inorder.size());
    }
};
```