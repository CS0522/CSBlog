---
title: 【刷题日记】二叉树-找树左下角的值-L513-Medium
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
date: 2024-10-31 16:38:04
---

给定一个二叉树的 根节点 root，请找出该二叉树的 最底层 最左边 节点的值。

假设二叉树中至少有一个节点。

<!-- more -->

---

## 思路

* 思路 1
    * 层序遍历，最后一层的第一个节点即为最后一层的最左边节点，因此记录遍历每层时的第一个节点。

## 学习点



## 代码

```cpp
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        // 层序遍历
        // 记录遍历的每层的第一个
        queue<TreeNode*> qe;
        qe.push(root);
        int first_of_layer = root->val;
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; ++i)
            {
                TreeNode *tmp = qe.front();
                qe.pop();

                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);

                // 记录每层第一个
                if (i == 0)
                    first_of_layer = tmp->val;
            }
        }

        return first_of_layer;
    }
};
```