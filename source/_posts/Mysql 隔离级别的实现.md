---
title: Mysql 隔离级别的实现
date: 2021/4/19
description: Mysql 隔离级别的实现
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

# Mysql 隔离级别的实现

> 以下内容都是基于 5.7 的 MySQL 的 innode 的存储引擎

## 一、并发问题的出现

- 读-读：没有并发问题

- 读-写：会造成数据不一致
- 写-写：造成数据丢失

## 二、当前读和快照读

## 三、MVCC 及其实现

## 四、乐观锁和悲观锁



