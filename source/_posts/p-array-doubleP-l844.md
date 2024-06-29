---
title: 【刷题日记】数组-比较含退格的字符串-L844-Easy
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
date: 2024-06-09 21:14:28
---

给定 s 和 t 两个字符串，当它们分别被输入到空白的文本编辑器后，如果两者相等，返回 true。# 代表退格字符。

<!-- more -->

---

[844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/description/)

## 思路

* 栈。时间空间复杂度较高
* 快慢指针。分别处理

## 学习点

* 在原始 `s` 和 `t` 上进行修改，只需要额外的 `O(1)` 空间复杂度和 `O(2*n)` 时间复杂度。快指针查找，慢指针写入

## 代码

<details>
  <summary>栈</summary>
  
  ```cpp
  // 栈
  class Solution {
  public:
      bool backspaceCompare(string s, string t) {
          string s_res = "";
          string t_res = "";
  
          stack<char> ss;
          stack<char> ts;
  
          for (int i = 0; i < s.size(); i++)
          {
              if (s[i] != '#')
              {
                  ss.push(s[i]);
              }
              else
              {
                  if (!ss.empty())
                      ss.pop();
              }
          }
  
          while (!ss.empty())
          {
              s_res += ss.top();
              ss.pop();
          }
          // reverse(s_res.begin(), s_res.end());
  
          for (int i = 0; i < t.size(); i++)
          {
              if (t[i] != '#')
              {
                  ts.push(t[i]);
              }
              else
              {
                  if (!ts.empty())
                      ts.pop();
              }
          }
  
          while (!ts.empty())
          {
              t_res += ts.top();
              ts.pop();
          }
          // reverse(t_res.begin(), t_res.end());
  
          return (s_res == t_res);
      }
  };
  ```

</details>

快慢指针

```cpp
// 快慢指针
class Solution {
public:
    bool backspaceCompare(string s, string t) {
        // 快慢指针
        int s_slow = 0;
        for (int fast = 0; fast < s.size(); ++fast)
        {
            // not '#'
            if (s[fast] != '#')
                s[s_slow++] = s[fast];
            // is '#'
            else
            {
                if (s_slow > 0)
                {
                    s_slow--;
                }
            }
        }
        // 新 s 长度为 s_slow

        int t_slow = 0;
        for (int fast = 0; fast < t.size(); ++fast)
        {
            // not '#'
            if (t[fast] != '#')
                t[t_slow++] = t[fast];
            // is '#'
            else
            {
                if (t_slow > 0)
                {
                    t_slow--;
                }
            }
        }
        // 新 t 长度为 t_slow

        // length is not equal
        if (s_slow != t_slow)
            return false;
        // length is equal
        else
        {
            for (int i = 0; i < s_slow; i++)
            {
                // 存在一个不一样
                if (s[i] != t[i])
                    return false;
            }
            return true;
        }
    }
};
```