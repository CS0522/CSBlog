---
title: 【刷题日记】二叉树-N叉树的最大深度-L559-Easy
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
date: 2024-10-30 14:57:44
---

给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

N 叉树输入按层序遍历序列化表示，每组子节点由空值分隔（请参见示例）。

<!-- more -->

---

## 思路

同 L104。

## 学习点



## 代码

```cpp
class Solution {
public:
    int maxDepth(Node* root) {
        int depth = 0;
        queue<Node*> qe;
        if (root)
            qe.push(root);
        while (!qe.empty())
        {
            int node_num = qe.size();
            for (int i = 0; i < node_num; i++)
            {
                auto tmp = qe.front();
                qe.pop();

                for (int j = 0; j < tmp->children.size(); ++j)
                {
                    if (tmp->children[j])
                        qe.push(tmp->children[j]);
                }
            }
            ++depth;
        }

        return depth;
    }
};
```