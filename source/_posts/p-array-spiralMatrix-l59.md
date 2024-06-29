---
title: 【刷题日记】数组-螺旋矩阵II-L59-Medium
tags:
  - C++
  - 刷题
  - 数组
  - 螺旋矩阵
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组 - 螺旋矩阵
comments: false
cover: false
date: 2024-06-10 20:40:12
---

给你一个正整数 n ，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的 n x n 正方形矩阵 matrix 。

<!-- more -->

---

## 思路

* 直接模拟过程，顺时针方向。用二维数组 `directions` 保存顺时针方向，当超过范围或遇到访问过元素时，改变方向

## 学习点

* `directions[0]、[1]、[2]、[3]` 分别表示 4 个方向，二维分别为 `row` 和 `col`

## 代码

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) 
    {
        // directions
        // 顺时针方向
        // 一维为某个方向，二维为 row 和 col
        int directions[4][2] = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
        // 最大的数
        int max_num = n * n;
        // results
        vector<vector<int>> spiral_metrix(n, vector<int>(n, 0));
        // 当前的数
        int curr_num = 1;
        int row = 0, col = 0;
        // 当前的方向
        int curr_direct = 0;
        // 按数填充
        while (curr_num <= max_num)
        {
            spiral_metrix[row][col] = curr_num;
            ++curr_num;
            // 计算下一个要填充的数的 row 和 col
            int next_row = row + directions[curr_direct][0];
            int next_col = col + directions[curr_direct][1];
            // 当超出范围或者碰到已经访问过的，则改变方向
            if (next_row >= n || next_col >= n || 
                next_row < 0 || next_col < 0 ||
                spiral_metrix[next_row][next_col] != 0)
            {
                // change direction
                curr_direct = (curr_direct + 1) % 4;
            }
            // 重新计算
            next_row = row + directions[curr_direct][0];
            next_col = col + directions[curr_direct][1];  

            row = next_row;
            col = next_col;
        } 

        return spiral_metrix;
    }
};
```