---
title: 【刷题日记】二叉树-二叉树的层平均值-L637-Easy
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
date: 2024-08-22 19:02:42
---

给定一个非空二叉树的根节点 root , 以数组的形式返回每一层节点的平均值。与实际答案相差 10-5 以内的答案可以被接受。

<!-- more -->

---

## 思路

* 二叉树层序遍历，计算每层的总和后结果集中压入平均值。

## 学习点



## 代码

```cpp
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        vector<double> res;
        if (!root)
            return res;
        queue<TreeNode*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            double sum_of_level = 0;
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                sum_of_level += tmp->val;

                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }
            res.push_back(sum_of_level / node_num * 1.0);
        }

        return res;
    }
};
```