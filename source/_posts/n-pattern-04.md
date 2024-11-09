---
title: 【学习笔记】设计模式（四）观察者模式
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
date: 2024-07-03 21:37:54
---

组件协作：观察者模式。

<!-- more -->

## 模式类型

组件协作

## 观察者模式

需要为某些对象建立一种“通知依赖关系”——一个对象的改变，所有依赖他的对象都得到通知。


## 使用背景

一个对象发生改变，所有依赖他的对象都要得到通知。

当一个对象的改变需要同时影响其他对象，并且不希望对象之间紧密耦合时，可以使用观察者模式。


## 举例

### 未使用观察者模式

```cpp
// FileSplitter
class FileSplitter
{
    // ...
    ProgressBar* m_progress_bar;
    
    FileSplitter(...) { ... }

    void split()
    {
        if (m_progress_bar != nullptr)
        {
            m_progress_bar->set_value(...);
        }
    }
};

// MainForm
class MainForm : public Form
{
    // ...
    ProgressBar* progress_bar;

    void button1_click()
    {
        // ...
        FileSplitter splitter(...);
        splitter.split();
    }
};
```


### 使用观察者模式

FileSplitter 类没有耦合界面类。

```cpp
// IProgress
class IProgress
{
    virtual void do_progress(float value) = 0;
    virtual ~do_progress() {}
};

// FileSplitter
class FileSplitter
{
    IProgress* m_iprogress; // 抽象通知

    FileSplitter(..., IProgress* iprogress) m_iprogress(iprogress) { ... }

    void split()
    {
        if (m_iprogress != nullptr)
        {
            m_iprogress->do_progress();
        }
    }
};

// MainForm
class MainForm : public Form, public IProgress
{
    // ...
    ProgressBar* progress_bar;

    void button1_click()
    {
        // ...
        FileSplitter splitter(..., this);
        splitter.split();
    }

    // 形式上看，FileSplitter 原先的 set_value 迁移到 MainForm 类中
    virtual void do_progress(float value)
    {
        progress_bar->set_value();
    }
};
```

另一个观察者模式的示例：

```java
// 被观察者的抽象基类
public abstract class Subject
{
    private IList<Observer> _observers = new List<Observer>(); // 当前主题对象的观察者集合
    // 添加观察者
    public void Attach(Observer observer)
    {
        _observers.Add(observer);
    }
    // 移除观察者
    public void Detach(Observer observer)
    {
        _observers.Remove(observer);
    }
    // 通知观察者
    public void Nofity()
    {
        Console.WriteLine("猫突然大叫一声...喵...");
        foreach (var item in _observers)
        {
            item.Update();
        }
    }
}

// 观察者的抽象基类
public abstract class Observer
{
    public abstract void Update();
}

// 被观察者的子类
public class Cat : Subject
{ }

// 观察者的子类1
public class Mouse : Observer
{
    public override void Update()
    {
        Console.WriteLine("老鼠：快跑，被老猫发现了...");
    }
}

// 观察者的子类2
public class Master : Observer
{
    public override void Update()
    {
        Console.WriteLine("主人：我家的猫又叫了，开灯看看怎么回事？");
    }
}

// 调用
internal class Client
{
    public void Start()
    {
        Cat cat = new Cat();
        cat.Attach(new Mouse());
        cat.Attach(new Master());
        cat.Nofity();
    }
}
```