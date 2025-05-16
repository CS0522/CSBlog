---
title: 【刷题日记】二叉树-合并二叉树-L617-Easy
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
date: 2025-05-16 20:22:55
---

给你两棵二叉树： root1 和 root2 。

想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，不为 null 的节点将直接作为新二叉树的节点。

返回合并后的二叉树。

注意: 合并过程必须从两个树的根节点开始。

<!-- more -->

---

## 思路

* 直接使用递归，递归终止条件为 传入的两个 root 节点均为空，即可返回；递归过程中，判断哪个 root 节点为空。非空节点递归构造其左子树、右子树，然后作为 root 节点返回。

## 学习点



## 代码

初始版本：

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2)
            return nullptr;
        if (root1 && root2)
        {
            root1->left = mergeTrees(root1->left, root2->left);
            root1->right = mergeTrees(root1->right, root2->right);
            root1->val += root2->val;
            return root1;
        }
        else
        {
            TreeNode* not_empty_root = root1 ? root1 : root2;
            not_empty_root->left = mergeTrees(not_empty_root->left, nullptr);
            not_empty_root->right = mergeTrees(not_empty_root->right, nullptr);
            return not_empty_root;
        }
    }
};
```

优化版本，其中一个节点为空的情况下完全可以合并，不需要单独罗列：

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2; // 如果t1为空，合并之后就应该是t2
        if (t2 == NULL) return t1; // 如果t2为空，合并之后就应该是t1
        // 修改了t1的数值和结构
        t1->left = mergeTrees(t1->left, t2->left);      // 左
        t1->right = mergeTrees(t1->right, t2->right);   // 右
        t1->val += t2->val;                             // 中
        return t1;
    }
};
```