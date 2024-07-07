---
title: 【刷题日记】哈希表-快乐数-L202-Easy
tags:
  - C++
  - 刷题
  - 哈希表
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 哈希表
comments: false
cover: false
date: 2024-07-07 00:16:50
---

编写一个算法来判断一个数 n 是不是快乐数。

<!-- more -->

---

## 思路

* 存在循环（重复）时不满足快乐数条件，返回 false。因此用哈希集合存储已经出现过的 sum 值。

## 学习点



## 代码

```cpp
class Solution {
public:
    long long cal_sum(int &n)
    {
        // 从个位开始保存
        long long sum = 0LL;
        while (n > 0)
        {
            int digit = n % 10;
            sum += digit * digit;
            n /= 10;
        }

        return sum;
    }

    bool isHappy(int n) {
        // 哈希表，如果存在相同值，说明存在循环，返回 false
        unordered_set<int> us;

        auto sum = cal_sum(n);
        while (sum != 1)
        {
            // 存在循环
            if (us.count(sum))
                return false;
            // 暂时没有循环
            us.insert(sum);
            n = sum;
            sum = cal_sum(n);
        }
        // 退出 while，说明满足 sum == 1
        return true;
    }
};
```