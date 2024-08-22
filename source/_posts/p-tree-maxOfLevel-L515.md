---
title: 【刷题日记】二叉树-在每个树行中找最大值-L515-Medium
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
date: 2024-08-22 19:20:27
---

给定一棵二叉树的根节点 root ，请找出该二叉树中每一层的最大值。

<!-- more -->

---

## 思路

* 层序遍历，记录每层的最大值。

## 学习点



## 代码

```cpp
class Solution {
public:
    vector<int> largestValues(TreeNode* root) {
        vector<int> res;
        if (!root)
            return res;
        queue<TreeNode*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            int max_of_level = qe.front()->val;

            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                if (max_of_level < tmp->val)
                    max_of_level = tmp->val;
                
                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }

            res.push_back(max_of_level);
        }

        return res;
    }
};
```