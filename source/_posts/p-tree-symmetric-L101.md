---
title: 【刷题日记】二叉树-对称二叉树-L101-Easy
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
date: 2024-10-30 14:32:52
---

给你一个二叉树的根节点 root ，检查它是否轴对称。（关于根节点轴对称）

<!-- more -->

---

## 思路

* 思路 1
    * 利用 L226 的翻转二叉树，将根节点的左子树翻转后与右子树一一比较节点，如果一样，则是关于根节点轴对称。根节点为空则不是轴对称。

* 思路 2
    * 不需要考虑翻转，直接一个遍历左边，一个遍历右边，然后比较是否相等。

## 学习点



## 代码

思路 1：

```cpp
class Solution {
public:
    // 比较两个树是否相同
    bool compareTree(TreeNode *tree1, TreeNode *tree2)
    {
        if (!tree1 && !tree2)
            return true;
        if (!tree1 || !tree2)
            return false;
        if (tree1->val != tree2->val)
            return false;
        return (compareTree(tree1->left, tree2->left) 
                && compareTree(tree1->right, tree2->right));
    }

    TreeNode *invertTree(TreeNode *root)
    {
        // 递归终止条件
        if (!root)
            return root;
        // 递归子树
        invertTree(root->left);
        invertTree(root->right);
            
        TreeNode *tmp_node = root->left;
        root->left = root->right;
        root->right = tmp_node;

        return root;
    }

    bool isSymmetric(TreeNode* root) {
        if (!root)
            return false;
        // 先反转左子树
        TreeNode *invertLeft = invertTree(root->left);
        // 比较左右子树是否一致
        bool res = compareTree(invertLeft, root->right);
        return res;
    }
};
```

思路 2：

```cpp
class Solution {
public:
    // 比较两个树是否相同
    bool compareTree(TreeNode *tree1, TreeNode *tree2)
    {
        if (!tree1 && !tree2)
            return true;
        if (!tree1 || !tree2)
            return false;
        if (tree1->val != tree2->val)
            return false;
        return (compareTree(tree1->left, tree2->right) 
                && compareTree(tree1->right, tree2->left));
    }

    bool isSymmetric(TreeNode* root) {
        if (!root)
            return false;
        // 比较两个树是否一致
        return compareTree(root->left, root->right);
    }
};
```