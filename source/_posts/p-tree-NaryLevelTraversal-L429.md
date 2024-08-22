---
title: 【刷题日记】二叉树-N叉树的层序遍历-L429-Medium
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
date: 2024-08-22 19:10:10
---

给定一个 N 叉树，返回其节点值的层序遍历。（即从左到右，逐层遍历）。

树的序列化输入是用层序遍历，每组子节点都由 null 值分隔（参见示例）。

<!-- more -->

---

## 思路

* 层序遍历，在加入下一层的子树时循环加入子树的节点。

## 学习点



## 代码

```cpp
class Node {
public:
    int val;
    vector<Node*> children;

    Node() {}

    Node(int _val) {
        val = _val;
    }

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};

class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<Node*> qe;
        qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            vector<int> level_res;
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                level_res.push_back(tmp->val);
                
                // 加入子树
                auto children_num = tmp->children.size();
                for (int j = 0; j < children_num; j++)
                {
                    if (tmp->children[j])
                        qe.push(tmp->children[j]);
                }
            }

            res.push_back(level_res);
        }

        return res;
    }
};
```