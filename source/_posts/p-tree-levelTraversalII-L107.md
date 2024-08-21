---
title: 【刷题日记】二叉树-二叉树的层序遍历II-L107-Medium
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
date: 2024-08-21 12:50:53
---

给你二叉树的根节点 root ，返回其节点值 自底向上的层序遍历 。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

<!-- more -->

---

## 思路

* 相比于 L102 的正序层序遍历，只需要将 `res` 的第一维（层级的顺序，而不是其中的元素的顺序）倒序即可。

## 学习点



## 代码

```cpp
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        // result
        vector<vector<int>> res;
        if (root == nullptr)
            return res;
        // queue
        queue<TreeNode*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            vector<int> level_traverse;
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();
                level_traverse.push_back(tmp->val);

                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }

            res.push_back(level_traverse);
        }

        // 相比于正序遍历只需要倒序第一维
        reverse(res.begin(), res.end());
        return res;
    }
};
```