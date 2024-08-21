---
title: 【刷题日记】二叉树-二叉树的层序遍历-L102-Medium
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
date: 2024-08-21 12:50:23
---

给你二叉树的根节点 root ，返回其节点值的 层序遍历 。 （即逐层地，从左到右访问所有节点）。

<!-- more -->

---

## 思路

* 层序遍历使用队列。题目需要按照不同层来输出，因此使用 `pair` 来组合 `TreeNode` 和 `levelIndex` 来标记节点的层级。
* 使用队列。使用 `size (queue.size())` 记录二叉树每层的节点数。每层遍历的时候都会弹空队列，然后加入下层的所有节点。这时候队列的 `size` 即为下一层的节点个数。 

## 学习点

* 用 `queue.size()` 记录每层的节点个数。

## 代码

我的：

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        // result
        vector<vector<int>> res;
        if (root == nullptr)
            return res;
        // <node, level>
        queue<pair<TreeNode*, int>> qe;
        qe.push(make_pair<>(root, 0));
        int curr_level = -1;
        while (!qe.empty())
        {
            auto p = qe.front();
            qe.pop();
            // 遍历到新的层
            if (curr_level != p.second)
            {
                vector<int> tmp;
                res.push_back(tmp);
            }
            // 加入结果集
            res[res.size() - 1].push_back(p.first->val);
            // 加入队列
            if (p.first->left)
                qe.push(make_pair<>(p.first->left, p.second + 1));
            if (p.first->right)
                qe.push(make_pair<>(p.first->right, p.second + 1));

            curr_level = p.second;
        }

        return res;
    }
};
```

第二个思路：

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        // result
        vector<vector<int>> res;
        if (root == nullptr)
            return res;
        // queue
        queue<TreeNode*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            vector<int> level_traverse;
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();
                level_traverse.push_back(tmp->val);

                if (tmp->left)
                    qe.push(tmp->left);
                if (tmp->right)
                    qe.push(tmp->right);
            }

            res.push_back(level_traverse);
        }

        return res;
    }
};
```