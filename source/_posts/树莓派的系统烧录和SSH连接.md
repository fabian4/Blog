---
title: 树莓派的系统烧录和SSH连接
date: 2020/11/23
description: 树莓派的系统烧录和SSH连接
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Xw5URiozpMkudvm.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Xw5URiozpMkudvm.png'
categories:
  - 树莓派
tags:
  - 树莓派
abbrlink: 47788
---

# 树莓派的系统烧录和SSH连接

## 一、SD卡的格式化和树莓派的系统烧录

由于收购的是二手树莓派，需要先将SD卡中原系统格式化，再安装一个新系统

这里我们使用 **SD Card Formatter** 将我们的SD卡进行格式化

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/JHkubf6o3nrBsci.png" alt="image-20201123201031272" style="zoom: 50%;" />



然后我们去树莓派的官网下载镜像系统 https://www.raspberrypi.org/software/operating-systems/

![image-20201123201647758](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/APOglBtu8YrG7fk.png)

由于我们没有屏幕，我们这里选择第三个没有桌面的树莓派系统镜像，使用 **Win32DiskImager** 来进行镜像烧录

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/E9F6kNbxDPrVQUy.png" alt="image-20201123202003369" style="zoom:50%;" />



## 二、WIFI配置

由于没有屏幕，这里我们选用 **SSH** 来连接我们的树莓派，为了获取树莓派的 **IP**，这里我们将自己的电脑和树莓派接入同一个网络。

- 在SD卡的文件目录下面新建一个`wpa_supplicant.conf` 文件

  ~~~conf
  country=cn
  update_config=1
  ctrl_interface=/var/run/wpa_supplicant
  
  network={
   ssid="网络1名称"
   psk="密码"
  }
  
  network={
   ssid="网络2名称"
   psk="密码"
  }
  
  network={
   ssid="网络3名称"
   psk="密码"
  }
  ~~~

- 新建一个名为 `SSH` 的无后缀文件，来让树莓派开启 SSH 服务

- 将电脑连接同一网络



## 三、SSH连接

给树莓派插上SD卡和电源线，启动树莓派

在终端命令行我们去 **ping raspberrypi** 就可以得到树莓派的IP地址了

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/5bYuDLl6vdKTOUA.png" alt="image-20201123204001165" style="zoom: 33%;" />

这里我们得到的是IPV6地址，我们可以通过xshell来进行远程连接我们的树莓派了

默认的树莓派登录：

- 用户名：pi
- 密码：raspberry

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Nv1fRBzpAoWYslk.png" alt="image-20201123204244035" style="zoom:50%;" />

现在我们就可以通过SSH来操控我们的树莓派了

