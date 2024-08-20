---
title: 【刷题日记】二叉树-前序遍历-L144-Easy
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
date: 2024-08-20 17:11:57
---

给你二叉树的根节点 root ，返回它节点值的 前序 遍历。

<!-- more -->

---

## 思路

* 递归法。
* 迭代法。

## 学习点

* 注意递归三要素：终止条件、返回值、递归操作

## 代码

递归法：

```cpp
struct TreeNode
{
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

#include <iostream>
#include <vector>
using namespace std;

class Solution
{
public:
    void preOrder(TreeNode *root, vector<int> &res)
    {
        // 递归终止
        if (root == nullptr)
            return;
        // 中左右
        res.push_back(root->val);
        preOrder(root->left, res);
        preOrder(root->right, res);
    }

    vector<int> preorderTraversal(TreeNode *root)
    {
        vector<int> res;
        preOrder(root, res);
        return res;
    }
};
```

迭代法：

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        stack<TreeNode*> st;
        vector<int> result;
        if (root == NULL) return result;
        st.push(root);
        while (!st.empty()) {
            TreeNode* node = st.top();
            st.pop();
            result.push_back(node->val);
            if (node->right) 
              st.push(node->right);            // 右（空节点不入栈）
            if (node->left) 
              st.push(node->left);             // 左（空节点不入栈）
        }
        return result;
    }
};
```