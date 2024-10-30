---
title: 【刷题日记】二叉树-完全二叉树的节点个数-L222-Easy
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
date: 2024-10-30 15:10:45
---

给你一棵 完全二叉树 的根节点 root ，求出该树的节点个数。

<!-- more -->

---

## 思路

层序遍历。

## 学习点



## 代码

```cpp
class Solution {
public:
    int countNodes(TreeNode* root) {
        queue<TreeNode*> qe;
        if (root)
            qe.push(root);
        // 计数
        int node_cnt = 0;
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; ++i)
            {
                TreeNode *tmp = qe.front();
                qe.pop();
                // 计数
                ++node_cnt;
                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }
        }

        return node_cnt;
    }
};
```