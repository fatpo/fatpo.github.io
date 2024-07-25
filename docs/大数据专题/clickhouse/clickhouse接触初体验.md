<!-- TOC -->

* [1. ClickHouse 初体验学习笔记](#1-clickhouse-初体验学习笔记)
    * [1.1. 前言](#11-前言)
    * [1.2. 环境准备](#12-环境准备)
        * [1.2.1. 安装ClickHouse](#121-安装clickhouse)
        * [1.2.2. 启动服务](#122-启动服务)
        * [1.2.3. 连接到ClickHouse](#123-连接到clickhouse)
    * [1.3. 基本操作](#13-基本操作)
        * [1.3.1. 创建数据库](#131-创建数据库)
        * [1.3.2. 创建表](#132-创建表)
        * [1.3.3. 插入数据](#133-插入数据)
        * [1.3.4. 查询数据](#134-查询数据)
        * [1.3.5. 更新数据](#135-更新数据)
        * [1.3.6. 删除数据](#136-删除数据)
    * [1.4. 性能优化](#14-性能优化)
        * [1.4.1. 使用合适的数据类型](#141-使用合适的数据类型)
        * [1.4.2. 使用合适的引擎](#142-使用合适的引擎)
        * [1.4.3. 分区和索引](#143-分区和索引)
    * [1.5. 常见问题及解决方法](#15-常见问题及解决方法)
        * [1.5.1. 无法启动服务](#151-无法启动服务)
        * [1.5.2. 性能问题](#152-性能问题)

<!-- TOC -->

# 1. ClickHouse 初体验学习笔记

## 1.1. 前言

ClickHouse是一款由Yandex开发的开源列式数据库管理系统，以其高性能和高压缩率而闻名，特别适用于实时分析处理海量数据。本文记录了我第一次使用ClickHouse的学习过程，包括安装、基本操作和一些常见问题的解决方法。

## 1.2. 环境准备

### 1.2.1. 安装ClickHouse

在Ubuntu系统上，可以通过以下命令安装ClickHouse：

```sh
sudo apt-get install apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4
echo "deb https://repo.clickhouse.com/deb/stable/ main/" | sudo tee /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
sudo apt-get install clickhouse-server clickhouse-client
```

### 1.2.2. 启动服务

安装完成后，启动ClickHouse服务：

```sh
sudo service clickhouse-server start
```

### 1.2.3. 连接到ClickHouse

使用ClickHouse客户端连接到服务器：

```sh
clickhouse-client
```

## 1.3. 基本操作

### 1.3.1. 创建数据库

```sql
CREATE
DATABASE my_database;
```

### 1.3.2. 创建表

在ClickHouse中创建表的过程与其他关系型数据库类似，但由于ClickHouse是`列式`存储的，定义表时需要指定引擎（如`MergeTree`）：

```sql
CREATE TABLE my_table
(
    id         UInt32,
    name       String,
    age        UInt8,
    created_at DateTime
) ENGINE = MergeTree()
ORDER BY id;
```

### 1.3.3. 插入数据

```sql
INSERT INTO my_table (id, name, age, created_at)
VALUES (1, 'Alice', 30, now());
INSERT INTO my_table (id, name, age, created_at)
VALUES (2, 'Bob', 25, now());
```

### 1.3.4. 查询数据

```sql
SELECT *
FROM my_table;
```

### 1.3.5. 更新数据

重要的说三遍：

* ClickHouse`不支持`直接的`UPDATE`操作，可以通过插入新数据来实现更新！
* ClickHouse`不支持`直接的`UPDATE`操作，可以通过插入新数据来实现更新！
* ClickHouse`不支持`直接的`UPDATE`操作，可以通过插入新数据来实现更新！

```sql
INSERT INTO my_table (id, name, age, created_at)
VALUES (1, 'Alice', 31, now());
```

### 1.3.6. 删除数据

重要的说三遍：

* ClickHouse也`不支持直接的DELETE操作`，可以通过`TTL机制`或`创建新表并复制需要的数据来实现删除`!
* ClickHouse也`不支持直接的DELETE操作`，可以通过`TTL机制`或`创建新表并复制需要的数据来实现删除`!
* ClickHouse也`不支持直接的DELETE操作`，可以通过`TTL机制`或`创建新表并复制需要的数据来实现删除`!

```sql
ALTER TABLE my_table DELETE WHERE id = 1;
```

## 1.4. 性能优化

### 1.4.1. 使用合适的数据类型

选择合适的数据类型可以显著提高性能和节省存储空间。例如，对于`年龄`字段，可以使用`UInt8`而不是`UInt32`。

### 1.4.2. 使用合适的引擎

ClickHouse支持多种存储引擎，选择合适的引擎可以优化查询性能。例如，`MergeTree引擎`适用于大多数场景，而`Log引擎`适用于简单的写入操作。

### 1.4.3. 分区和索引

分区和索引可以显著提高查询性能。可以使用`PARTITION BY`和`ORDER BY`子句来定义分区和排序键。

```sql
CREATE TABLE my_table
(
    id         UInt32,
    name       String,
    age        UInt8,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(created_at)
ORDER BY id;
```

## 1.5. 常见问题及解决方法

### 1.5.1. 无法启动服务

检查配置文件是否正确，日志文件是否有错误信息：

```sh
sudo tail -f /var/log/clickhouse-server/clickhouse-server.log
```

### 1.5.2. 性能问题

使用EXPLAIN命令分析查询计划，检查是否有不必要的全表扫描：

```sql
EXPLAIN
SELECT *
FROM my_table
WHERE age = 30;
```
