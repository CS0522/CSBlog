---
title: 【刷题日记】二叉树-左叶子之和-L404-Easy
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
date: 2024-10-31 15:34:56
---

给定二叉树的根节点 root ，返回所有左叶子之和。

<!-- more -->

---

## 思路

* 思路 1
    * 用一个标记来标识当前节点是否为左子节点（通过上一个父节点传递标记参数）。如果为左子节点且已经没有了左右子节点，则该左子节点为左叶子节点。

* 思路 2
    * 不使用标记。通过父节点来判断父节点的左子节点是否为叶子节点。

## 学习点

* 使用标记来标识。

## 代码

思路 1：

```cpp
class Solution {
public:
    void cal_left_leaves(TreeNode *root, int &res, bool is_left)
    {
        if (!root)
            return;
        cal_left_leaves(root->left, res, true);
        cal_left_leaves(root->right, res, false);
        // 叶子节点且为左
        if (!root->left && !root->right && is_left)
            res += root->val;
    }

    int sumOfLeftLeaves(TreeNode* root) {
        int res = 0;
        cal_left_leaves(root, res, false);

        return res;
    }
};
```

思路 2：

```cpp
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        if (root == NULL) return 0;
        if (root->left == NULL && root->right== NULL) return 0;

        int leftValue = sumOfLeftLeaves(root->left);    // 左
        if (root->left && !root->left->left && !root->left->right) { // 左子树就是一个左叶子的情况
            leftValue = root->left->val;
        }
        int rightValue = sumOfLeftLeaves(root->right);  // 右

        int sum = leftValue + rightValue;               // 中
        return sum;
    }
};
```