---
title: 【刷题日记】二叉树-二叉树的所有路径-L257-Easy
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
date: 2024-10-31 10:51:08
---

给你一个二叉树的根节点 root ，按任意顺序，返回所有从根节点到叶子节点的路径。

叶子节点 是指没有子节点的节点。

<!-- more -->

---

## 思路

* 思路 1
    * 通过结果集来获取当前的遍历路径。按照深度优先遍历方法，当遍历到一个节点后，就弹出队列尾部的路径，然后加上当前节点的路径。队列尾部最后一个路径就是当前正在遍历的路径。之后如果左分支或右分支存在，则将当前路径压入队列；如果左右分支都不存在，也需要将当前路径压入队列。
    * 问题在于存在多次的 vector 的 push_back 和 pop_back 操作。

* 思路 2
    * 通过函数参数传递当前的遍历路径。将回溯隐藏在函数参数 `path + "->"` 中，这样可以使左子树遍历完毕后，`path` 回溯到当前节点，然后继续遍历右子树。

## 学习点



## 代码

思路 1：

```cpp
class Solution {
public:
    void getPaths(TreeNode* root, vector<string> &res)
    {
        // 递归终止条件
        if (!root)
            return;
        string curr_path;
        // 取出当前路径
        if (!res.size())
            curr_path = to_string(root->val);
        else
        {
            curr_path = res[res.size() - 1] + "->" + to_string(root->val);
            res.pop_back();
        }
        // 压入当前路径
        if (root->left)
        {
            res.push_back(curr_path);
            getPaths(root->left, res);
        }
        if (root->right)
        {
            res.push_back(curr_path);
            getPaths(root->right, res);
        }
        if (!root->left && !root->right)
            res.push_back(curr_path);
    }

    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> res;
        getPaths(root, res);

        return res;
    }
};
```

思路 2：

```cpp
class Solution {
private:

    void traversal(TreeNode* cur, string path, vector<string>& result) {
        path += to_string(cur->val); // 中
        if (cur->left == NULL && cur->right == NULL) {
            result.push_back(path);
            return;
        }
        // 回溯隐藏在 path + "->" 中
        if (cur->left) 
            traversal(cur->left, path + "->", result); // 左
        if (cur->right) 
            traversal(cur->right, path + "->", result); // 右
    }

public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        string path;
        if (root == NULL) return result;
        traversal(root, path, result);
        return result;

    }
};
```