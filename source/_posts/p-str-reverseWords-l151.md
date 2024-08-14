---
title: 【刷题日记】字符串-反转单词-L151-Medium
tags:
  - C++
  - 刷题
  - 字符串
toc: true
languages:
  - zh-CN
categories:
  - 刷题日记
  - 字符串
comments: false
cover: false
date: 2024-08-14 17:58:37
---

给你一个字符串 s ，请你反转字符串中 单词 的顺序。

单词 是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的 单词 分隔开。

返回 单词 顺序颠倒且 单词 之间用单个空格连接的结果字符串。

注意：输入字符串 s中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

<!-- more -->

---

## 思路

* 用栈存储。遍历完后弹出栈，添加到新字符串并加入空格。
* 双指针法移除空格，之后反转字符串（O(1)空间复杂度）。

## 学习点

* 双指针移除空格。

## 代码

栈：

```cpp
class Solution {
public:
    string reverseWords(string s) {
        // 用栈
        stack<string> st;
        string tmp = "";
        for (int i = 0; i < s.length(); i++)
        {
            // 遇到空格压入栈
            if (s[i] == ' ')
            {
                if (tmp != "")
                    st.push(tmp);
                tmp = "";
            }
            // 非空格更新临时字符串
            else
                tmp += s[i];
        }
        // 最后一个单词压入栈
        if (tmp != "")
            st.push(tmp);

        // 输出
        string res = "";
        while (!st.empty())
        {
            res += st.top();
            st.pop();
            // 为了最后一个单词尾部没有空格
            if (!st.empty())
                res += " ";
        }
        
        return res;
    }
};
```

双指针：(O(1)复杂度)

```cpp
class Solution {
public:
    void reverse(string& s, int start, int end){ //翻转，区间写法：左闭右闭 []
        for (int i = start, j = end; i < j; i++, j--) {
            swap(s[i], s[j]);
        }
    }

    void removeExtraSpaces(string& s) {//去除所有空格并在相邻单词之间添加空格, 快慢指针。
        int slow = 0;   //整体思想参考https://programmercarl.com/0027.移除元素.html
        for (int i = 0; i < s.size(); ++i) { //
            if (s[i] != ' ') { //遇到非空格就处理，即删除所有空格。
                if (slow != 0) s[slow++] = ' '; //手动控制空格，给单词之间添加空格。slow != 0说明不是第一个单词，需要在单词前添加空格。
                while (i < s.size() && s[i] != ' ') { //补上该单词，遇到空格说明单词结束。
                    s[slow++] = s[i++];
                }
            }
        }
        s.resize(slow); //slow的大小即为去除多余空格后的大小。
    }

    string reverseWords(string s) {
        removeExtraSpaces(s); //去除多余空格，保证单词之间之只有一个空格，且字符串首尾没空格。
        reverse(s, 0, s.size() - 1);
        int start = 0; //removeExtraSpaces后保证第一个单词的开始下标一定是0。
        for (int i = 0; i <= s.size(); ++i) {
            if (i == s.size() || s[i] == ' ') { //到达空格或者串尾，说明一个单词结束。进行翻转。
                reverse(s, start, i - 1); //翻转，注意是左闭右闭 []的翻转。
                start = i + 1; //更新下一个单词的开始下标start
            }
        }
        return s;
    }
};
```