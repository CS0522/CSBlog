---
title: 【学习笔记】设计模式（三）策略模式
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
date: 2024-07-03 16:54:57
---

设计模式之策略模式。

<!-- more -->

## 策略模式

软件构建过程中，某些对象使用的算法可能多种多样，经常改变。如何在运行时根据需要透明地改变对象的算法，将算法与对象本身解耦？

遵循**开放封闭原则**，开放扩展（类的增加），封闭修改（原对象的修改）。

定义一系列算法，把它们一个个封装起来，并使他们能够在**运行时**根据需要发生切换（变化），这个模式使得算法可以独立于使用它的客户程序而变化（扩展，子类化）。

接口保持稳定，其中的算法发生变化。


## 使用场景

含有许多 if-else 条件语句的代码通常可以使用策略模式。绝对稳定不变的 if-else 可以不用使用策略模式。


## 举例

### 未使用策略模式

```cpp
enum TaxBase
{
    CN_Tax,
    US_Tax,
    DE_Tax
};

class SalesOrder
{
    double cal_tax()
    {
        if (CN_Tax) { // A 
        }
        else if (US_Tax) { // B
        }
        else if (DE_Tax) { // C 
        }
    }
};
```

### 使用策略模式

```cpp
class TaxStrategy
{
    virtual double cal_tax(const Context& context) = 0;
    virtual ~TaxStrategy() {}
};

class CNTax : public TaxStrategy
{
    virtual double cal_tax(const Context& context)
    {
        // A
    }
};

class USTax : public TaxStrategy
{
    ...
};

class DETax : public TaxStrategy
{
    ...
};


class SalesOrder
{
    TaxStrategy* strategy;

    SalesOrder(StrategyFactory* strategyFactory)
    {
        // 工厂模式，是哪一个子类由工厂决定
        this->strategy = strategyFactory->newStrategy();
    }
    ~SalesOrder() { delete strategy; }

    calculate_tax()
    {
        // ...
        Context context();
        // ...
        double val = strategy->cal_tax(context);
        // ...
    }
};
```