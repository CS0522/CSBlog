---
title: 【刷题日记】数组-移动零-L283-Easy
tags:
  - C++
  - 刷题
  - 数组
  - 双指针
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组 - 双指针
comments: false
cover: false
date: 2024-06-09 17:27:10
---

给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
请注意，必须在不复制数组的情况下原地对数组进行操作。

<!-- more -->

---

[283. 移动零](https://leetcode.cn/problems/move-zeroes/solutions/489622/yi-dong-ling-by-leetcode-solution/)

## 思路

* 局部左右指针，实际上存在很多不必要的交换和比较，复杂度为`O(n^2)`

**update：快慢指针**

* 慢指针指向当前已经处理好的序列的尾部，快指针指向待处理序列的头部
* 慢指针写入，快指针查找
* 当快指针所指元素不为 0，交换快慢指针元素（交换是因为慢指针指的元素可能不为 0）


## 学习点

* 快慢指针

## 代码

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int len = nums.size();
        // 局部左右指针
        int left = 0;
        while (left < len)
        {
            // left != 0
            // left 指针前进
            if (nums[left] != 0)
            {
                left++;
            }
            // left == 0
            // right 指针从 left + 1 开始查找，
            // 找到第一个不为 0 的元素，并交换
            else
            {
                int right = left + 1;
                while (right < len)
                {
                    if (nums[right] == 0)
                    {
                        right++;
                    }
                    else
                    {
                        // 找到第一个不为 0 的元素
                        break;
                    }
                }
                if (right >= len)
                {
                    // 一直到结束都是 0，所有比较已完毕
                    break;
                }
                // 交换
                nums[left++] = nums[right];
                nums[right] = 0;
            }
        }
    }
};
```

**update：快慢指针**

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int len = nums.size();
        // 快慢指针
        int slow = 0;
        int fast = 0;
        while (fast < len)
        {
            // 右边不是 0，交换
            if (nums[fast] != 0)
            {
                swap(nums[slow], nums[fast]);
                slow++;
            }
            fast++;
        }
    }
};
```