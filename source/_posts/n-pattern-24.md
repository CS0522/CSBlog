---
title: 【学习笔记】设计模式（二十四）解释器
tags:
  - 设计模式
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 设计模式
comments: false
cover: false
date: 2024-11-09 19:24:15
---

领域规则：解释器

<!-- more -->

## 模式类型

领域规则

## 解释器

某些变化虽然频繁，但可以抽象为某种规则。提供了评估语言的语法或表达式的方式。

解释器模式给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。

关键在于 Terminal Expression 和 Non-Terminal Expression，定义语言的文法结构。

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-pattern-24/expr1.png)

## 使用场景

* 编译器
* 正则表达式
* SQL 解析

## 举例

### 使用解释器模式

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-pattern-24/expr2.png)

```cpp
// 抽象表达式类
class Expression
{
public:
    virtual int interpreter(map<char, int> var_map) = 0;
};

// 变量表达式
// 终端表达式
class VarExpression : public Expression
{
private:
    char var;
public:
    VarExpression(const char &var)
    {
        this->var = var;
    }
    int interpreter(map<char, int> var_map) override
    {
        return var_map[var];
    }
};

// 符号表达式
// 非终端表达式
class SymbolExpression : public Expression
{
protected:
    // 两个操作数
    Expression *left;
    Expression *right;
public:
    SymbolExpression(Expression *left, Expression *right) : left(left), right(right) { }
};

class AddExpression : public SymbolExpression
{
public:
    AddExpression(Expression *left, Expression *right) : SymbolExpression(left, right) { }
    
    int interpreter(map<char, int> var_map) override
    {
        return (left->interpreter(var_map) + right->interpreter(var_map));
    }
};

class SubExpression : public SymbolExpression
{
public:
    AddExpression(Expression *left, Expression *right) : SymbolExpression(left, right) { }
    
    int interpreter(map<char, int> var_map) override
    {
        return (left->interpreter(var_map) - right->interpreter(var_map));
    }
};

Expression *analyize(map<char, int> var_map, string expr)
{
    int expr_len = expr.length();
    Expression *left = nullptr, *right = nullptr;
    stack<Expression*> st;
    for (int i = 0; i < expr_len; ++i)
    {
        switch (expr[i])
        {
            case '+':
                left = st.top();
                right = new VarExpression(expr[++i]);
                st.push(new AddExpression(left, right));
                break;
            case '-';
                left = st.top();
                right = new VarExpression(expr[++i]);
                st.push(new SubExpression(left, right));
                break;
            default:
                st.push(new VarExpression(expr[i]));
                break;
        }
    }

    Expression* expr_final = st.top();
    return expr_final;
}

int main()
{
    string expr = "a+b-c+d";
    map<char, int> var_map;
    var_map.insert(make_pair<>('a', 5));
    var_map.insert(make_pair<>('b', 6));
    var_map.insert(make_pair<>('c', 7));
    var_map.insert(make_pair<>('d', 8));

    Expression *expr_final = analyize(expr, var_map);
    int res = expr_final->interpreter(var_map);

    cout << res << endl;

    return 0;
}
```