---
title: 【刷题日记】二叉树-最大二叉树-L654-Medium
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
date: 2025-05-15 22:05:26
---

给定一个不重复的整数数组 nums 。 最大二叉树 可以用下面的算法从 nums 递归地构建:

创建一个根节点，其值为 nums 中的最大值。
递归地在最大值 左边 的 子数组前缀上 构建左子树。
递归地在最大值 右边 的 子数组后缀上 构建右子树。

<!-- more -->

---

## 思路

递归函数中，首先找到当前数组中的最大元素，根节点通过 new 出来；然后该元素左边子数组构造左子树，右边子数组构造右子树。递归终止条件为传入的 vector 长度为 0。

## 学习点



## 代码

初始版本：

```cpp
class Solution {
public:
    // @return max index
    int findMaxIndex(vector<int>& nums)
    {
        int max_index = 0;
        int max_num = nums[0];
        for (int i = 0; i < nums.size(); i++)
        {
            if (nums[i] > max_num)
            {
                max_index = i;
                max_num = nums[i];
            }
        }
        return max_index;
    }

    void constructTree(TreeNode* &root, vector<int>& sub_nums)
    {
        if (!sub_nums.size())
            return;
        // 1. 找到当前数组中最大的元素
        int max_index = findMaxIndex(sub_nums);
        if (root == nullptr)
            root = new TreeNode(sub_nums[max_index]);
        vector<int> left_sub_nums(sub_nums.begin(), sub_nums.begin() + max_index);
        vector<int> right_sub_nums(sub_nums.begin() + max_index + 1, sub_nums.end());
        // 2. 左数组构造左子树
        constructTree(root->left, left_sub_nums);
        // 3. 右数组构造右子树
        constructTree(root->right, right_sub_nums);
    }

    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        TreeNode* root = nullptr;
        constructTree(root, nums);
        return root;
    }
};
```

优化版本，避免多次构造 vector 导致的时间和空间浪费：

```cpp
class Solution {
public:
    // @return max index
    int findMaxIndex(vector<int>& nums, 
                    const int& start, const int& end)
    {
        int max_index = start;
        int max_num = nums[start];
        for (int i = start; i < end; i++)
        {
            if (nums[i] > max_num)
            {
                max_index = i;
                max_num = nums[i];
            }
        }
        return max_index;
    }

    void constructTree(TreeNode* &root, vector<int>& nums, 
                        const int& start, const int& end)
    {
        if (!(end - start))
            return;
        // 1. 找到当前数组中最大的元素
        int max_index = findMaxIndex(nums, start, end);
        if (root == nullptr)
            root = new TreeNode(nums[max_index]);
        // 2. 左数组构造左子树
        constructTree(root->left, nums, start, max_index);
        // 3. 右数组构造右子树
        constructTree(root->right, nums, max_index + 1, end);
    }

    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        TreeNode* root = nullptr;
        constructTree(root, nums, 0, nums.size());
        return root;
    }
};
```