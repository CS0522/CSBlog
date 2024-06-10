---
title: 【刷题日记】数组-水果成篮-L904-Medium
tags:
  - C++
  - 刷题
  - 数组
  - 滑动窗口
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 数组
  - 滑动窗口
comments: false
cover: false
date: 2024-06-10 15:30:02
---

你正在探访一家农场，农场从左到右种植了一排果树。这些树用一个整数数组 fruits 表示，其中 fruits[i] 是第 i 棵树上的水果 种类 。

你想要尽可能多地收集水果。然而，农场的主人设定了一些严格的规矩，你必须按照要求采摘水果。

<!-- more -->

---

[904. 水果成篮](https://leetcode.cn/problems/fruit-into-baskets/)

## 思路

* 采用滑动窗口。考虑：窗口内是啥？什么时候移动 `start`？什么时候移动`end`？
* 刚开始不成熟的想法是，`end` 初始指向 `start` 后一个，然后依次遍历；窗口内是目标元素；当 `count > 2`，移动 `start`。想法与正解差不多，但问题出在 `start` 该向右移动多少元素的地方。我的想法就是简单的当 `start != start + 1`，（即当满足这个条件后 `count` 就减少 1）再移动一次后停止了，这样会导致移动的位置过少，譬如：`[3,3,3,1,2,1,1,2,3,3,4]`，问题体现在当 `end` 指向 `fruits[8] = 3` 时，通过我的想法 `start` 只会前进到 `fruits[4] = 2` 上进而停止了。正确的位置应该是 `fruits[7] = 2` 上。所以正确的停止条件是当 unordered_map 中记录的键值对减少为 2 后，才停止

## 学习点

* 用哈希表 `unordered_map` 记录键值对`(ele, count)`
* `unordered_map` 与 `map` 相比不需要排序
* `unordered_map` 的增删改查：
  
  ```cpp
  //插入
	votes["小明"]++;    //直接添加（值为int类型时才这么用）
	votes["李华"]++;    //当不存在该key时，会自动添加该新项
	votes["小明"]++;    //当已经存在该key时，则直接对value进行自增
	pair<string, int> vote1("小方", 4);    //新建单个pair
	votes.insert(vote1);    //插入创建的pair
	votes.emplace("陈一", 7);    //效果同insert，但是votes.insert("陈一", 7)会报错
	votes.insert(make_pair<string, int>("张三", 3));

  //查找
  auto vote3 = votes.find("王五");
  if (vote3 == votes.end())    //等于end表示没有找到该key
		cout << "没找到" << endl;
	else
		cout << "找到 " << vote3->first << ": " << vote3->second << endl;

  //删除
  votes.erase("张三");    //通过key删除
	votes.erase(votes.begin());    //通过位置删除
	votes.erase(vote3);    //通过迭代器删除，这里，vote3即上面查找的王五

  //修改
  votes["刘二"] = 3;    //修改方式1
	votes.at("李四") = 6;    //修改方式2
  ```

## 代码

错误答案：

<details>
    <summary>我的错误想法</summary>

    ```cpp
      class Solution {
        public:
          int totalFruit(vector<int>& fruits) {
              int n = fruits.size();
              int start = 0, end = start + 1;
              int maxLen = 1;
              // 记录窗口内不同种类的个数
              int count = 1;
              // 记录窗口内是否存在该种类
              vector<bool> exist(n, false);
              exist[fruits[start]] = true;
              while (end < n && start < end) {
                  if (fruits[end] != fruits[end - 1]) {
                      // 如果end指向的元素不存在
                      if (!exist[fruits[end]]) {
                          count++;
                          exist[fruits[end]] = true;
                      }
                      // 新加这个种类后种类大于2
                      /**
                      * 这里有问题
                       */
                      // start前进
                      while (count > 2) {
                          if (fruits[start] != fruits[start + 1]) {
                              count--;
                              exist[fruits[start]] = false;
                          }
                          start++;
                      }
                  }
                  maxLen = max(maxLen, end - start + 1);
                  end++;
              }

              return maxLen;
          }
      };
    ```
    
</details>


正确答案：

```cpp
// 滑动窗口
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int n = fruits.size();
        // 对应种类和出现次数
        unordered_map<int, int> count;

        int start = 0;
        int maxLen = 0;
        for (int end = 0; end < n; ++end)
        {
            // 加入当前键值
            ++count[fruits[end]];
            // 当种类超过2
            while (count.size() > 2)
            {
                // 迭代器取得fruits中最左边的数
                // 删除这个元素，start指针对应前进
                auto it = count.find(fruits[start]);
                --it->second;
                if (it->second == 0)
                {
                    count.erase(it);
                }
                ++start;
            }
            maxLen = max(maxLen, end - start + 1);
        }
        return maxLen;
    }
};
```