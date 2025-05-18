---
title: 【刷题日记】二叉树-验证二叉搜索树-L98-Medium
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
date: 2025-05-18 17:00:27
---

给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。

有效 二叉搜索树定义如下：

节点的左子树只包含 小于 当前节点的数。
节点的右子树只包含 大于 当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

<!-- more -->

---

## 思路

* 递归验证每个节点。在验证每个节点时，验证左子树、右子树，然后在判断 root 节点时，获取左子树最大值、右子树最小值，比较 root 和它们的大小；

* 二叉搜索树具有 中序遍历为从小到大顺序 的特点，对 BST 进行中序遍历，然后检查数组是否按序。

## 学习点



## 代码

初始版本：

```cpp
class Solution {
public:
    int getMaxVal(TreeNode *root)
    {
        TreeNode *p = root;
        while (p->right)
            p = p->right;
        return p->val;
    }
    int getMinVal(TreeNode *root)
    {
        TreeNode *p = root;
        while (p->left)
            p = p->left;
        return p->val;
    }
    bool isValidBST(TreeNode* root) {
        bool is_left_valid = true;
        bool is_right_valid = true;
        bool is_root_valid = true;
        int left_max_val, right_min_val;
        
        if (root->left)
        {
            is_left_valid = isValidBST(root->left);
            left_max_val = getMaxVal(root->left);
        }
        if (root->right)
        {
            is_right_valid = isValidBST(root->right);
            right_min_val = getMinVal(root->right);
        }

        if (root->left)
            is_root_valid = is_root_valid && (root->val > left_max_val);
        if (root->right)
            is_root_valid = is_root_valid && (root->val < right_min_val);
        
        return (is_left_valid && is_right_valid && is_root_valid);
    }
};
```

优化版本：

```cpp
class Solution {
public:
    void traversalBST(TreeNode* root, vector<int>& nodes)
    {
        if (!root)
            return;
        // 中序遍历
        traversalBST(root->left, nodes);
        nodes.push_back(root->val);
        traversalBST(root->right, nodes);
    }
    bool isValidBST(TreeNode* root)
    {
        vector<int> nodes;
        traversalBST(root, nodes);
        for (size_t i = 1; i < nodes.size(); i++)
        {
            if (nodes[i - 1] >= nodes[i])
                return false;
        }
        return true;
    }
};
```