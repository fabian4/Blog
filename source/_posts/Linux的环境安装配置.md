---
title: Linux的环境安装配置
date: 2020/10/8
description: Linux的环境安装配置
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/fMZLAIWDh5yGlwd.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/fMZLAIWDh5yGlwd.jpg'
categories:
  - Linux
tags:
  - Linux
  - 环境配置
abbrlink: 51116
---

# Linux的环境安装配置

​	本文以腾讯云 **CentOS 7.6** 系统为例进行服务器环境配置和软件安装

## 一、配置 yum 阿里镜像源

> Yum(全称为 Yellow dogUpdater, Modified)是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于[RPM](https://baike.so.com/doc/1772529-1874476.html)包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

1. 备份你的原镜像文件，以免出错后可以恢复。

   ~~~bash
   mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
   ~~~

2. 下载新的CentOS-Base.repo 到/etc/yum.repos.d/

   ~~~bash
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   ~~~

3. 运行yum makecache生成缓存

   ~~~bash
   yum clean all
   yum makecache
   ~~~

4. 更新 yum

   ~~~bash
   yum update
   ~~~

## 二、GIT ⼯具安装

~~~bash
yum install git
~~~

检测安装

~~~bash
[root@VM-4-5-centos ~]# git version
git version 1.8.3.1
~~~

## 三、JDK（JAVA环境）安装

~~~bash
# 查询所有 JDK 依赖
yum search java | grep -i --color jdk
# 选择版本安装
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
~~~

~~~bash
# 检查安装
[root@VM-4-5-centos ~]# java -version
openjdk version "1.8.0_272"
OpenJDK Runtime Environment (build 1.8.0_272-b10)
OpenJDK 64-Bit Server VM (build 25.272-b10, mixed mode)
~~~

## 四、node 环境安装

~~~bash
# yum仓库版本太低了，这里我们去官网下载最新版
curl --silent --location https://rpm.nodesource.com/setup_14.x | bash -
# 开始安装
yum install nodejs
~~~

~~~bash
# 检查安装
[root@VM-4-5-centos ~]# node
Welcome to Node.js v14.15.1.
Type ".help" for more information.
> .exit
[root@VM-4-5-centos ~]# npm version
{
  npm: '6.14.8',
  ares: '1.16.1',
  brotli: '1.0.9',
  cldr: '37.0',
  icu: '67.1',
  llhttp: '2.1.3',
  modules: '83',
  napi: '7',
  nghttp2: '1.41.0',
  node: '14.15.1',
  openssl: '1.1.1g',
  tz: '2020a',
  unicode: '13.0',
  uv: '1.40.0',
  v8: '8.4.371.19-node.17',
  zlib: '1.2.11'
}
~~~

## 五、python 环境安装

~~~bash
# 查看 python 版本
[root@VM-4-5-centos ~]# python
Python 2.7.5 (default, Nov 16 2020, 22:23:17) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[root@VM-4-5-centos ~]# python3
Python 3.6.8 (default, Nov 16 2020, 16:55:22) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
# 这里默认安装了 python 2.7 和 python 3.6
~~~

~~~bash
# 安装 python 2
yum install python
# 安装 python 3
yum install python3
~~~

## 六、Mysql 数据库安装

### 1. 下载并安装MySQL官方的 Yum Repository

~~~bash
# 下载安装用的Yum Repository
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
# 开始用 yum 安装
yum install mysql57-community-release-el7-10.noarch.rpm
yum install mysql-community-server
~~~

### 2.  MySQL数据库设置

~~~bash
# 先启动mysql
systemctl start  mysqld.service
# 关闭命令
systemctl stop  mysqld.service
# 查看启动状态
systemctl status mysqld.service

[root@VM-4-5-centos ~]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-10-07 11:52:33 CST; 49s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 705 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 645 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 708 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─708 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Dec 04 11:52:28 VM-4-5-centos systemd[1]: Starting MySQL Server...
Dec 04 11:52:33 VM-4-5-centos systemd[1]: Started MySQL Server.

# 此时还有一个问题，就是因为安装了Yum Repository，以后每次yum操作都会自动更新，需要把这个卸载掉
yum remove mysql57-community-release-el7-10.noarch
~~~

~~~bash
# 此时MySQL已经开始正常运行，不过要想进入MySQL还得先找出此时root用户的密码，通过如下命令可以在日志文件中找出密码
grep "password" /var/log/mysqld.log

2020-10-07T03:52:30.076236Z 1 [Note] A temporary password is generated for root@localhost: Ac:y#Dtoc0O2

# 输入密码 Ac:y#Dtoc0O2 进入mysql
[root@VM-4-5-centos ~]# mysql -uroot -pAc:y#Dtoc0O2
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.32

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
# 输入初始密码，此时不能做任何事情，因为MySQL默认必须修改密码之后才能操作数据库：
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '数据库密码';

# 这里有个问题，新密码设置的时候如果设置的过于简单会报错
# 原因是因为MySQL有密码设置的规范，具体是与validate_password_policy的值有关
# 解决方法就是修改密码为规范复杂的密码
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)
mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)

# 操作完成上面的，现在还不能用可视化的客户端进行连接，需要我们进行授权
mysql> grant all on *.* to root@'%' identified by '数据库密码';
Query OK, 0 rows affected, 1 warning (0.00 sec)
~~~

> 可视化工具的登录如 Navicat 等连接不上去，可以去看一看系统的防火墙，或者服务器的管理处如腾讯云阿里云配置网关

## 七、Redis 缓存数据库安装

~~~bash
# yum 安装redis
yum install redis
# 启动 redis
service redis start
# 关闭 redis
service redis stop
# 查看 redis 状态
service redis status
# 进入 redis 命令行
redis-cli
~~~

## 八、Nginx 服务器安装

~~~bash
# 添加 Nginx 到 yum 源
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# 安装 Nginx
yum install nginx
# 启动 Nginx
systemctl start nginx.service
# 关闭 Nginx
systemctl stop nginx.service
~~~



