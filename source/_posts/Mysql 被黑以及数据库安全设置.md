---
title: Mysql 被黑以及数据库安全设置
date: 2020/12/7
description: Mysql 被黑以及数据库安全设置
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sCxZTUefziHIhw8.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sCxZTUefziHIhw8.png'
categories:
  - Mysql
tags:
  - Mysql
  - Linux
  - 安全
abbrlink: 28312
---

# Mysql 被黑以及数据库安全设置

## 一、数据库被黑

某天接到消息，部署在测试服务器上的后台所有接口全部报错。
于是我登上了服务器数据库，发现测试数据表全部被删，只留下了一张 `WARNING` 表

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Z3wXnAvTeoLS7G5.png)

发现自己也居然被 比特币勒索了

## 二、数据库安全

### 1. 数据库端口

尽量修改常用的 3306 端口，改为不常用的其他端口

打开文件my.cnf,然后增加端口参数，设定端口，注意该端口应是未被使用的，注意一定要先保存再退出

然后重启数据库

### 2. root权限和账号密码

root账号密码尽量复杂，且所有其他人都发放其他不具有root权限的账号

~~~bash
# 只允许指定ip连接
create user '新用户名'@'localhost' identified by '密码';
# 允许所有ip连接（用通配符%表示）
create user '新用户名'@'%' identified by '密码';
# 给新用户授权 基本格式如下
grant all privileges on 数据库名.表名 to '新用户名'@'指定ip' identified by '新用户密码' ;
# 允许访问所有数据库下的所有表
grant all privileges on *.* to '新用户名'@'指定ip' identified by '新用户密码' ;
# 指定数据库下的指定表
grant all privileges on test.test to '新用户名'@'指定ip' identified by '新用户密码' ;
# 设置用户拥有所有权限也就是管理员
grant all privileges on *.* to '新用户名'@'指定ip' identified by '新用户密码' WITH GRANT OPTION;
# 拥有查询权限
grant select on *.* to '新用户名'@'指定ip' identified by '新用户密码' WITH GRANT OPTION;
# 其它操作权限说明,select查询 insert插入 delete删除 update修改
# 设置用户拥有查询插入的权限
grant select,insert on *.* to '新用户名'@'指定ip' identified by '新用户密码' WITH GRANT OPTION;
# 取消用户查询的查询权限
REVOKE select ON what FROM '新用户名';
# 删除用户
DROP USER username@localhost;
# 修改后刷新权限
FLUSH PRIVILEGES;
~~~



