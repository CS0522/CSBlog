---
ttitle: 【刷题日记】栈和队列-前 K 个高频元素-L347-Medium
tags:
  - C++
  - 刷题
  - 栈和队列
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 栈和队列
comments: false
cover: false
date: 2024-08-17 21:21:19
---

给你一个整数数组 nums 和一个整数 k ，请你返回其中出现频率前 k 高的元素。你可以按 任意顺序 返回答案。

<!-- more -->

---

## 思路

* 使用 `map` 键值对存储每个元素出现的次数；然后按照出现的次数进行从大到小排序（排序可以用小顶堆优化）；最后取前 k 个元素。
* 看了下官解确实是用小顶堆的。

## 学习点

* 为什么使用小顶堆（什么时候用小顶堆，什么时候用大顶堆）。
* 函数指针 `auto cmp = [] (...) { ... };` 以及 `delctype()`。

## 代码

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        map<int, int> ele_freq;

        // 用 map 键值对先存储每个元素对应的出现次数
        for (int i = 0; i < nums.size(); i++)
            ele_freq[nums[i]]++;

        // map 不能根据 value 排序
        // 使用 vector 接收 map 数据，并排序
        vector<pair<int, int>> map_eles;
        for(auto it = ele_freq.begin(); it != ele_freq.end(); it++)
        {
            map_eles.emplace_back(pair<int, int>(it->first, it->second));
        }
        
        auto cmp = [] (pair<int, int> &a, pair<int, int> &b) { return a.second > b.second; };
        // 应该不需要 sort，只需要前 k 个即可
        // 可以使用 k-小顶堆：当大于 top 元素，弹出堆顶后入堆（堆顶元素为当前 k 个中的最小值，即选出的 k 个最大值中的最小值）
        // 最后堆中元素为最大的 k 个
        sort(map_eles.begin(), map_eles.end(), cmp);

        vector<int> res;
        for (int i = 0; i < k; i++)
        {
            res.push_back(map_eles[i].first);
        }

        return res;
    }
};
```

使用小顶堆：

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) 
    {
        // 要统计元素出现频率
        unordered_map<int, int> map; // map<nums[i],对应出现的次数>
        for (int i = 0; i < nums.size(); i++) {
            map[nums[i]]++;
        }

        // 对频率排序
        // 定义一个小顶堆，大小为k
        auto cmp = [] (pair<int, int> &a, pair<int, int> &b) { return a.second > b.second; };
        priority_queue<pair<int, int>, vector<pair<int, int>>, delctype(cmp)> pri_que;

        // 用固定大小为k的小顶堆，扫面所有频率的数值
        for (unordered_map<int, int>::iterator it = map.begin(); it != map.end(); it++) {
            pri_que.push(*it);
            if (pri_que.size() > k) { // 如果堆的大小大于了K，则队列弹出，保证堆的大小一直为k
                pri_que.pop();
            }
        }

        // 找出前K个高频元素，因为小顶堆先弹出的是最小的，所以倒序来输出到数组
        vector<int> result(k);
        for (int i = k - 1; i >= 0; i--) {
            result[i] = pri_que.top().first;
            pri_que.pop();
        }
        return result;
    }
};
```