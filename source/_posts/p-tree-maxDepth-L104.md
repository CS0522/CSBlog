---
title: 【刷题日记】二叉树-二叉树的最大深度-L104-Easy
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
date: 2024-08-22 22:51:58
---

给定一个二叉树 root ，返回其最大深度。

二叉树的 最大深度 是指从根节点到最远叶子节点的最长路径上的节点数。

<!-- more -->

---

## 思路

* 层序遍历，每次遍历一层后，depth 计数加 1。

## 学习点



## 代码

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        int depth = 0;
        queue<TreeNode*> qe;
        if (root)
            qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }
            ++depth;
        }

        return depth;
    }
};
```