---
title: 【学习笔记】设计模式（八）抽象工厂
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
date: 2024-07-09 11:59:38
---

对象创建：抽象工厂。

<!-- more -->

## 模式类型

对象创建

## 抽象工厂

提供一个接口，让该接口负责创建一系列相关或者相互依赖的对象，无需指定他们具体的类。

## 使用场景

有应对“多系列对象构建”的需求变化。“系列对象”指的是在某一特定系列下的对象之间有相互依赖、或作用的关系。不同系列的对象之间不能相互依赖。

主要应对“新系列”的需求变动。

系列对象如：

* SQL Server 系列：SQLConnection、SQLCommand、SQLReader

* MySQL 系列：MySQLConnection、MySQLCommand、MySQLReader

## 举例

### 未使用

这里的对象都写死了，当需求的数据库从 SQL Server 变更为 MySQL 时，修改起来很麻烦。

```cpp
class EmployeeDAO {
public:
    vector<EmployeeDAO> getEmployees() {
        SQLConnection *connection = new SQLConnection();
        
        SQLCommand *command = new SQLCommand();
        command->setConnection(connection);

        SQLReader *reader = command->executeReader();
        ...
    }
};
```

### 使用抽象工厂

因为三个接口类必须都是同一个系列，如果其中某个是其他系列则报错。所以三个接口类可以合成一个工厂类，同一个工厂保证了关联性。

```cpp
// 接口类
class IDBConnection {
    virtual fn() = 0;
};

class IDBCommand {
    virtual fn() = 0;
};

class IDBReader {
    virtual fn() = 0;
};

// 工厂类
// class IDBConnectionFactory {
//     virtual createConnection() = 0;
// };

// class IDBCommandFactory {
//     virtual createCommand() = 0;
// };

// class IDBReaderFactory {
//     virtual createReader() = 0;
// };

// 三个工厂类合成一个工厂类
class IDBFactory {
    virtual createConnection() = 0;
    virtual createCommand() = 0;
    virtual createReader() = 0;
};

// 具体实现类
class SQLConnection : public IDBConnection {

};

class SQLCommand : public IDBCommand {

};

class SQLReader : public IDBReader {

};

// 具体工厂类
// class SQLConnectionFactory : public IDBConnectionFactory {

// };

// class SQLCommandFactory : public IDBCommandFactory {

// };

// class SQLReaderFactory : public IDBReaderFactory {

// };

class SQLFactory : public IDBFactory {

};

class OracleConnection : public IDBConnection {

};

class OracleCommand : public IDBCommand {

};

class OracleReader : public IDBReader {

};

// 具体工厂类
class OracleFactory : public IDBFactory {

};

class EmployeeDAO {
    IDBFactory *dbFactory;

public:
    vector<EmployeeDAO> getEmployees() {
        IDBConnection *connection = dbFactory->createConnection();
        
        IDBCommand *command = dbFactory->createCommand();
        command->setConnection(connection);

        IDBReader *reader = command->executeReader();
        ...
    }
};
```