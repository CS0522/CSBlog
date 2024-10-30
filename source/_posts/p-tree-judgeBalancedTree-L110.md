---
title: 【刷题日记】二叉树-平衡二叉树-L110-Easy
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
date: 2024-10-30 15:12:53
---

给定一个二叉树，判断它是否是平衡二叉树

<!-- more -->

---

## 思路

* 思路 1
    * 直接递归判断左子树和右子树是否是平衡二叉树，然后再判断以当前节点为根节点的树是否平衡
    * 问题在于时间复杂度高，存在子树的高度多次计算的情况

* 思路 2
    * 将判断的逻辑加入到 `calHight()` 函数中，当计算得到非平衡时，设置特殊返回值，从而进行剪枝。

## 学习点

* 当其中一个子树为非平衡时，设置特殊返回值，进行剪枝从而减少计算开销。

## 代码

思路 1：

```cpp
class Solution {
public:
    int calHight(TreeNode* root) {
        if (!root)
            return 0;
        return (max(calHight(root->left), calHight(root->right)) + 1);
    }

    bool isBalanced(TreeNode* root) {
        if (!root)
            return true;
        // 分别递归判断左子树和右子树是否平衡，然后判断当前树是否平衡
        if (isBalanced(root->left) && isBalanced(root->right))
        {
            return (fabs(calHight(root->left) - calHight(root->right)) <= 1);
        }
        return false;
    }
};
```

思路 2：

```cpp
class Solution {
public:
    int calHight(TreeNode* root) {
        if (!root)
            return 0;
        int left_hight = calHight(root->left);
        int right_hight = calHight(root->right);
        if (left_hight == -1 || right_hight == -1)
            return -1;
        return (fabs(left_hight - right_hight) <= 1 ? 
                max(left_hight, right_hight) + 1 :
                -1);
    }

    bool isBalanced(TreeNode* root) {
        return (calHight(root) != -1);
    }
};
```