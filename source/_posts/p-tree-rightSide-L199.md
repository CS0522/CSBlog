---
title: 【刷题日记】二叉树-二叉树的右视图-L199-Medium
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
date: 2024-08-22 15:31:47
---

给定一个二叉树的 根节点 root，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

<!-- more -->

---

## 思路

* 二叉树的层序遍历，但仅仅在结果集中加入每层的最后一个节点的值。
* 若考虑通过深度遍历去遍历右子树，但从 右视图 视角来看，如果右子树为空，则右视图会看到左子树的最右边的节点，因此这个思路不正确。

## 学习点



## 代码

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

#include <iostream>
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        // 层序遍历，只弹出最后一个节点
        vector<int> res;
        if (root == nullptr)
            return res;
        queue<TreeNode*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                if (i == node_num - 1)
                    res.push_back(tmp->val);
                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }
        }

        return res;
    }
};
```