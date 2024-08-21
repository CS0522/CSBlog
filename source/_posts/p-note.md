---
title: 【刷题日记】记录题型思路
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
date: 2024-06-09 21:06:05
---

记录下遇到的题型，可以首先往哪个方向去考虑。

<!-- more -->

---

## 数组

### 二分查找

* {% post_link p-array-erfen-l704 %}  

* {% post_link p-array-erfen-l34 %}  

* {% post_link p-array-erfen-l69 %}

#### 循环边界条件

通过区间来判定：

* `[left, right]`: while(left <= right), left = mid + 1, right = mid - 1

* `[left, right)`: while(left < right), left = mid + 1, right = mid

* `(left, right)`: while (left < right - 1), left = mid, right = mid


### 双指针

* {% post_link p-array-doubleP-l26 %}  

* {% post_link p-array-doubleP-l27 %}  

* {% post_link p-array-doubleP-l283 %}  

* {% post_link p-array-doubleP-l844 %}  

* {% post_link p-array-doubleP-l977 %}


#### 快慢指针

* 快指针：用于遍历，查找元素

* 慢指针：用于写入，更新元素。题目一般需要在这一步进行不一样的处理

#### 左右指针

注意从两边向中间指针移动的时候，最后的边界条件。可以看在最后一次处理的时候，数组的所有元素是否都已经处理完毕


### 滑动窗口

* {% post_link p-array-moveWindow-l209 %}

* {% post_link p-array-moveWindow-l904 %}

与指针有点类似。窗口长度固定或者不固定。看作一个区间，就是不断的调节子序列的起始位置和终止位置，从而得出我们要想的结果。

实现滑动窗口，主要确定如下三点：
  * 窗口内是什么？
  * 如何移动窗口的起始位置？
  * 如何移动窗口的结束位置？


### 螺旋矩阵

* {% post_link p-array-spiralMatrix-l59 %}

直接模拟，注意循环条件，找到循环时同一的操作。


## 链表

### 创建虚拟头节点

* {% post_link p-linklist-basic-l707 %}  

* {% post_link p-linklist-delNFromEnd-l19 %}  

* {% post_link p-linklist-swap-l24 %}  

在操作当前节点必须要找前一个节点才能操作的情况时，就会导致遇到头节点的时候，需要进行特殊的处理。

因此，对头节点前加入虚拟节点后，可以使得头节点和其他的节点进行统一的处理，不需要单独进行处理。

### 新建节点并移动指针

* {% post_link p-linklist-reverse-l206 %}  

`ans = new ListNode(x.val, ans)`，当前指向节点A（或 null），构建新节点 B，其 next 指针指向节点 A（或 null），同时指针指向新节点 B。

### 栈存储指针

* {% post_link p-linklist-delNFromEnd-l19 %}  

* {% post_link p-linklist-reverse-l206 %}  

栈存在先进后出的特性，如果需要倒着数链表，可以尝试使用栈。

### 哈希表 unordered_set 存储指针

* {% post_link p-linklist-intersection-l160 %} 

* {% post_link p-linklist-circle-l142 %}  

哈希表存储已出现过的指针，用 `count(p)` 函数找出已出现过的元素。


## 哈希表

**需要查询一个元素是否出现过，或者一个元素是否在集合里的时候，就要第一时间想到哈希法！！**

### target - x

* {% post_link p-hash-2sum-l1 %} 

* {% post_link p-hash-4sum-l454 %}  

与正常查找 `x + y = target` 不同，先用哈希表存储已经出现过的元素；然后再遍历数组，查找当前元素 `x` 的 `target - x`。

## 字符串匹配

### KMP 算法

见到一次就背一次，多背。


## 栈与队列

### 用栈的情况

* 先进后出，后进先出
* 如果操作数据的方向与原始数据遍历的方向相反，考虑用栈

### 用队列的情况

* 先进先出，后进后出
* 队列的模板用法：
  ```cpp
  qe.push(a);
  while (!qe.empty())
  {
    auto tmp = qe.front();
    qe.pop();
    // 处理
    // ...
    qe.push(...);
  }
  ```


## 递归

递归三要素：

* 确定递归函数的参数和返回值：确定哪些参数是递归的过程中需要处理的，那么就在递归函数里加上这个参数， 并且还要明确每次递归的返回值是什么进而确定递归函数的返回类型。

* 确定终止条件：写完了递归算法, 运行的时候，经常会遇到栈溢出的错误，就是没写终止条件或者终止条件写的不对，操作系统也是用一个栈的结构来保存每一层递归的信息，如果递归没有终止，操作系统的内存栈必然就会溢出。

* 确定单层递归的逻辑：确定每一层递归需要处理的信息。在这里也就会重复调用自己来实现递归的过程。

## 二叉树

### 二叉树的深度遍历

使用（栈）递归法，或者迭代法。递归法更简单。

### 二叉树的广度（层序）遍历

使用队列，模板：

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> que;
        if (root != NULL) que.push(root);
        vector<vector<int>> result;
        while (!que.empty()) {
            int size = que.size();
            vector<int> vec;
            // 这里一定要使用固定大小 size，不要使用 que.size()，因为 que.size 是不断变化的
            for (int i = 0; i < size; i++) {
                TreeNode* node = que.front();
                que.pop();
                vec.push_back(node->val);
                if (node->left) que.push(node->left);
                if (node->right) que.push(node->right);
            }
            result.push_back(vec);
        }
        return result;
    }
};
```

### 