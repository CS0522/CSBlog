---
title: 【刷题日记】二叉树-二叉搜索树中的众数-L501-Easy
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
date: 2025-10-23 08:55:47
---

给你一个含重复值的二叉搜索树（BST）的根节点 root ，找出并返回 BST 中的所有 众数（即，出现频率最高的元素）。

如果树中有不止一个众数，可以按 任意顺序 返回。

假定 BST 满足如下定义：

* 结点左子树中所含节点的值 小于等于 当前节点的值
* 结点右子树中所含节点的值 大于等于 当前节点的值
* 左子树和右子树都是二叉搜索树

<!-- more -->

---

## 思路

* 用数组保存每个值对应的出现频率。数组下标为值，数组的元素为每个值对应的出现频率

* 用指针保存前一个树中前一个节点，与当前节点进行比较。如果相同则计数频率增加，不同则计数重置，并判断是否需要加入结果列表

## 学习点



## 代码

思路 1：

```cpp
class Solution {
public:
    int val_freq[200001] = {0};
    void traverseBST(TreeNode *root) {
        if (root == nullptr)
            return;
        traverseBST(root->left);
        val_freq[(100000 + root->val)] += 1;
        traverseBST(root->right);
    }
    vector<int> findMode(TreeNode* root) {
        vector<int> res;
        traverseBST(root);
        // find max
        int max_freq = 0;
        for (int i = 0; i < 200001; i++)
        {
            if (val_freq[i] >= max_freq)
            {
                if (val_freq[i] != max_freq)
                    res.clear();
                res.push_back(i - 100000);
                max_freq = val_freq[i];
            }
        }
        return res;
    }
};
```

思路 2：

```cpp
lass Solution {
private:
    int maxCount = 0; // 最大频率
    int count = 0; // 统计频率
    TreeNode* pre = NULL;
    vector<int> result;
    void searchBST(TreeNode* cur) {
        if (cur == NULL) return ;

        searchBST(cur->left);       // 左
                                    // 中
        if (pre == NULL) { // 第一个节点
            count = 1;
        } else if (pre->val == cur->val) { // 与前一个节点数值相同
            count++;
        } else { // 与前一个节点数值不同
            count = 1;
        }
        pre = cur; // 更新上一个节点

        if (count == maxCount) { // 如果和最大值相同，放进result中
            result.push_back(cur->val);
        }

        if (count > maxCount) { // 如果计数大于最大值频率
            maxCount = count;   // 更新最大频率
            result.clear();     // 很关键的一步，不要忘记清空result，之前result里的元素都失效了
            result.push_back(cur->val);
        }

        searchBST(cur->right);      // 右
        return ;
    }

public:
    vector<int> findMode(TreeNode* root) {
        count = 0;
        maxCount = 0;
        pre = NULL; // 记录前一个节点
        result.clear();

        searchBST(root);
        return result;
    }
};
```