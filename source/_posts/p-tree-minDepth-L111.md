---
title: 【刷题日记】二叉树-二叉树的最小深度-L111-Easy
tags:
  - C++
  - 刷题
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
comments: false
cover: false
date: 2024-08-22 22:52:09
---

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

<!-- more -->

---

## 思路

* 层序遍历，当出现首个没有左、右子树的节点时，该节点为首个叶子节点，其拥有最短深度。

## 学习点



## 代码

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
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

                // 如果没有左右子树，则该节点为叶子节点，为最短深度
                if (!tmp->left && !tmp->right)
                    return depth + 1;
                else
                {
                    if (tmp->left)
                        qe.push(tmp->left);
                    if (tmp->right)
                        qe.push(tmp->right);
                }
            }
            ++depth;
        }

        return depth;
    }
};
```