---
title: 【刷题日记】二叉树-中序遍历-L94-Easy
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
date: 2024-08-20 17:12:23
---

给你二叉树的根节点 root ，返回它节点值的 中序 遍历。

<!-- more -->

---

## 思路

* 递归法。
* 迭代法。

## 学习点

* 迭代法需要特殊处理。需要**额外定义一个指针**指向正在处理的节点，否则在回溯节点的时候，会重复遍历之前访问过的节点。

## 代码

递归法：

```cpp
class Solution
{
public:
    void inOrder(TreeNode *root, vector<int> &res)
    {
        // 递归终止
        if (root == nullptr)
            return;
        // 左中右
        inOrder(root->left, res);
        res.push_back(root->val);
        inOrder(root->right, res);
    }

    vector<int> inorderTraversal(TreeNode *root)
    {
        vector<int> res;
        inOrder(root, res);
        return res;
    }
};
```

迭代法：

```cpp
class Solution
{
public:
    void inOrder(TreeNode *root, vector<int> &res)
    {
        // 递归终止
        if (root == nullptr)
            return;
        // 左中右
        inOrder(root->left, res);
        res.push_back(root->val);
        inOrder(root->right, res);
    }

    vector<int> inorderTraversal(TreeNode *root)
    {
        // vector<int> res;
        // inOrder(root, res);
        // return res;

        // 迭代法
        vector<int> res;
        stack<TreeNode*> st;
        TreeNode* curr = root;
        while (curr != nullptr || !st.empty())
        {
            // 左
            if (curr != nullptr)
            {
                st.push(curr);
                curr = curr->left;
            }
            // 中、右
            else
            {
                curr = st.top();
                st.pop();
                res.push_back(curr->val);
                curr = curr->right;
            }
        }

        return res;
    }
};
```