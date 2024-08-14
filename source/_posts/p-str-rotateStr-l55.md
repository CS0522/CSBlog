---
title: 【刷题日记】字符串-右旋字符串-L55-Easy
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
date: 2024-08-14 17:59:08
---

字符串的右旋转操作是把字符串尾部的若干个字符转移到字符串的前面。给定一个字符串 s 和一个正整数 k，请编写一个函数，将字符串中的后面 k 个字符移到字符串的前面，实现字符串的右旋转操作。 

例如，对于输入字符串 "abcdefg" 和整数 2，函数应该将其转换为 "fgabcde"。

<!-- more -->

---

## 思路

* 用额外空间临时存储子字符串。
* 多次局部反转。

## 学习点

* 局部字符串反转。

## 代码

子字符串：

```cpp
#include <iostream>
using namespace std;

int main()
{
    int k;
    string s;
    while (cin >> k)
    {
        cin >> s;
        // 用临时字符串保存后面 k 个字符
        string last_k = s.substr(s.length() - k, k);
        // 用临时变量保存前面的字符
        string first_sub = s.substr(0, s.length() - k);
        // 拼接
        string res = last_k + first_sub;
        
        cout << res << endl;
    }
    
    return 0;
}
```

局部反转：

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
int main() {
    int n;
    string s;
    cin >> n;
    cin >> s;
    int len = s.size(); //获取长度

    reverse(s.begin(), s.end()); // 整体反转
    reverse(s.begin(), s.begin() + n); // 先反转前一段，长度n
    reverse(s.begin() + n, s.end()); // 再反转后一段

    cout << s << endl;

}
```