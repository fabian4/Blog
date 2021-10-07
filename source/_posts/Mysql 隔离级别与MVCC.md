---
title: Mysql 隔离级别与MVCC
date: 2021/4/19
description: Mysql 隔离级别与MVCC
top_img: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CuscoCathedral_ZH-CN9834821723_1920x1080.jpg
cover: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CuscoCathedral_ZH-CN9834821723_1920x1080.jpg
categories:
  - Mysql
tags:
  - Mysql
  - MVCC
abbrlink: 10729
---

# Mysql 隔离级别与MVCC

> 以下内容都是基于 5.7 的 MySQL 的 innode 的存储引擎

## 一、问题的出现

- 读-读：没有并发问题
- 读-写：会造成数据不一致  
- 写-写：造成数据丢失问题

**很明显在两个事务同时读操作，是不存在线程并发安全问题。但当一旦开始出现了写操作，那么就会开始干扰另一个事务了。**

为了解决 **读-写** 和 **写-写** 这两个场景下的问题，innodb 引入了 **MVCC** 和 **锁机制** 来保证事务的隔离

## 二、隔离级别

这里我们引入一个场景再来看看事务的隔离级别

~~~sql
create table T(c int) engine=InnoDB;
insert into T(c) values(1);
~~~

|     事务A      |     事务B      |
| :------------: | :------------: |
|    启动事务    |                |
| 查询得到结果 1 |                |
|                |    启动事务    |
|                | 查询得到结果 1 |
|                |  将 1 改成 2   |
|  查询得到值 a  |                |
|                |    提交事务    |
|  查询得到值 b  |                |
|    提交事务    |                |
|  查询得到值 c  |                |

**在不同隔离级别下 a b c 的值：**

- **读未提交** `Read UnCommitted`：

  - a：2
  - b：2
  - c：2

  事务A 总是能直接读到 事务B 未提交的内容，那么查询到的就一直是最新的数据

- **读已提交** `Read Committed`：

  - a：1
  - b：2
  - c：2

  事务A 只能读到 事务B 已提交的内容，那么在提交之后就能读到新的值

- **可重复读** `Repeatable Read`：

  - a：1
  - b：1
  - c：2

  事务A 在可重复读 的隔离级别下，能保证在同一个事务中读到的值是不受其他事务影响的

- **串行化** `Serializable`：

  - a：1
  - b：1
  - c：2

  在串行化的隔离级别下，事务B 的修改操作会被 事务A 阻塞住，这样知道 事务A 提交才会继续执行，所以 事务A 查询之后任然是初始的值

> 在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。
>
> - 在“可重复读”隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。
> - 在“读提交”隔离级别下，这个视图是在每个 SQL 语句开始执行的时候创建的。
> - 这里需要注意的是，“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念；
> - 而“串行化”隔离级别下直接用加锁的方式来避免并行访问。

我们可以通过 show variables 来查看当前的隔离级别

~~~bash
mysql> show variables like 'transaction_isolation';
+-----------------------+----------------+
| Variable_name | Value |
+-----------------------+----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+----------------+
~~~

## 二、MVCC 及 实现


