---
title: 【刷题日记】二叉树-后序遍历-L145-Easy
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
date: 2024-08-20 17:12:15
---

给你二叉树的根节点 root ，返回它节点值的 后序 遍历。

<!-- more -->

---

## 思路

* 递归法。
* 迭代法。

## 学习点

* 迭代法与前序遍历顺序部分相反，前序为 中左右，倒序后为 右左中，则在前序遍历入栈的顺序改为先入左子树后入右子树即可。

## 代码

递归法：

```cpp
class Solution
{
public:
    void postOrder(TreeNode *root, vector<int> &res)
    {
        if (root == nullptr)
            return;
        // 左右中
        postOrder(root->left, res);
        postOrder(root->right, res);
        res.push_back(root->val);
    }

    vector<int> postorderTraversal(TreeNode *root)
    {
        vector<int> res;
        postOrder(root, res);
        return res;
    }
};
```

迭代法：

```cpp
class Solution
{
public:
    void postOrder(TreeNode *root, vector<int> &res)
    {
        if (root == nullptr)
            return;
        // 左右中
        postOrder(root->left, res);
        postOrder(root->right, res);
        res.push_back(root->val);
    }

    vector<int> postorderTraversal(TreeNode *root)
    {
        // vector<int> res;
        // postOrder(root, res);
        // return res;

        // 迭代法
        vector<int> res;
        stack<TreeNode*> st;
        if (root == nullptr)
            return;
        st.push(root);
        while (!st.empty())
        {
            auto p = st.top();
            st.pop();
            res.push_back(p->val);
            // 空值不入栈
            if (p->left) 
                st.push(p->left);
            if (p->right) 
                st.push(p->right);
        }

        reverse(res.begin(), res.end());
        return res;
    }
};
```