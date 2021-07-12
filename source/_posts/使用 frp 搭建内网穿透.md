---
title: 使用 frp 搭建内网穿透
date: 2021/3/5
description: 使用 frp 搭建内网穿透
top_img: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/img-4807e805e5208cd813eeb4870650afcc.jpg
cover: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/img-4807e805e5208cd813eeb4870650afcc.jpg
categories:
  - 教程
tags:
  - frp
abbrlink: 22217
---

# 使用 frp 搭建内网穿透

## 一、什么是 frp

通过在具有公网 IP 的节点上部署 frp 服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：

- 客户端服务端通信支持 TCP、KCP 以及 Websocket 等多种协议。
- 采用 TCP 连接流式复用，在单个连接间承载更多请求，节省连接建立时间。
- 代理组间的负载均衡。
- 端口复用，多个服务通过同一个服务端端口暴露。
- 多个原生支持的客户端插件（静态文件查看，HTTP、SOCK5 代理等），便于独立使用 frp 客户端完成某些工作。
- 高度扩展性的服务端插件系统，方便结合自身需求进行功能扩展。
- 服务端和客户端 UI 页面。

> 1. 在办公室访问家里的电脑，反之亦然 
> 2. 自己电脑上的项目，方便发给客户朋友演示。比如我做了个小网站，发给朋友看看未上线版本，发个url给他就好了。
> 3. 调试一些需要远程调用的程序，远程调用比如微信的API 回调接口。



## 二、服务端搭建

1. https://github.com/fatedier/frp/releases 下载对应安装包

2. 修改配置文件 `frps.ini`

   ~~~ini
   [common]
   bind_port = 7000  
   
   max_pool_count = 5
   max_ports_per_client = 0
   authentication_timeout = 900
   ~~~

3. 启动项目

   ~~~shell
   ./frps -c ./frps.ini 
   ~~~

## 三、客户端搭建

1. https://github.com/fatedier/frp/releases 下载对应安装包

2. 修改配置文件 `frpc.ini`

   ~~~ini
   [common]
   server_addr = 公网ip地址
   server_port = 7000
   [ssh]
   type = tcp
   local_ip = 本地地址
   local_port = 本地端口
   remote_port = 远程端口
   ~~~

   common 为通用配置

   - server_addr 为公网服务器 IP 地址（云服务器IP）
   - server_port 为公网服务器配置的端口

   ssh 用于终端命令行访问

   - type 连接类型，默认为 tcp
   - local_ip 本地 IP 
   - local_port 用于 ssh 的端口号，默认 22
   - remote_port 映射的服务端端口，访问该端口时默认转发到客户端的 22 端口

3. 启动项目

   ~~~shell
   ./frpc -c ./frpc.ini 
   ~~~



## 四、文件结构和配置详情

### 1. 文件结构

- frpc                    # 客户端二进制文件 
- frpc_full.ini           # 客户端配置文件完整示例 
- frpc.ini                # 客户端配置文件 
- frps                    # 服务端二进制文件 
- frps_full.ini           # 服务端配置文件完整示例 
- frps.ini              # 服务端配置文件

### 2. 服务端配置

~~~ini
[common]                        # 通用配置段
bind_addr = 0.0.0.0             # 绑定的IP地址，支持IPv6，不指定默认0.0.0.0；
bind_port = 7000                # 服务端口；
bind_udp_port = 7001            # 是否使用udp端口，不使用删除或注释本行；
kcp_bind_port = 7000            # 是否使用kcp协议，不使用删除或注释本行；
# proxy_bind_addr = 127.0.0.1   # 代理监听地址，默认和bind_addr相同；

# 虚拟主机
vhost_http_port = 80            # 是否启用虚拟主机，端口可以和bind_port相同；
vhost_https_port = 443
vhost_http_timeout = 60         # 后端虚拟主机响应超时时间，默认为60s；

# 开启frps仪表盘可以检查frp的状态和代理的统计信息。
dashboard_addr = 0.0.0.0        # frps仪表盘绑定的地址；
dashboard_port = 7500           # frps仪表盘绑定的端口；
dashboard_user = admin          # 访问frps仪表盘的用户；     
dashboard_pwd = admin           # 密码；
assets_dir = ./static           # 仪表盘页面文件目录，只适用于调试；

# 日志配置文件
log_file = ./frps.log           # 日志文件,不指定日志信息默认输出到控制台；
log_level = info                # 日志等级，可用等级“trace, debug, info, warn, error”；
log_max_days = 3                # 日志保存最大保存时间；

token = 12345678                # 客户端与服务端通信的身份验证令牌

heartbeat_timeout = 90          # 心跳检测超时时间，不建议修改默认配置，默认值为90；？

# 指定允许客户端使用的端口范围，未指定则没有限制；
allow_ports = 2000-3000,3001,3003,4000-50000

max_pool_count = 5              # 每个客户端连接服务端的最大连接数；
max_ports_per_client = 0        # 每个客户端最大可以使用的端口，0表示无限制

authentication_timeout = 900    # 客户端连接超时时间（秒），默认为900s；

subdomain_host = frps.com       # 自定义子域名，需要在dns中将域名解析为泛域名；

tcp_mux = true                  # 是否使用tcp复用，默认为true；
                                # frp只对同意客户端的连接进行复用；
~~~

### 3. 客户端配置

~~~ini
[common]                        # 通用配置段

server_addr = 0.0.0.0           # server的IP地址；支持IPv6
server_port = 7000              # server的端口；

# 如果要通过http或socks5代理连接frps，可以在此处或在全局环境变量中设置代理，只支持tcp协议；
# http_proxy = http://user:passwd@192.168.1.128:8080
# http_proxy = socks5://user:passwd@192.168.1.128:1080

# 客户端日志
log_file = ./frpc.log       # 指定日志文件；
log_level = info            # 指定日志等级；
log_max_days = 3


token = 12345678            # 客户端与服务端通信的身份验证令牌

# 设置管理地址，用于通过http api控制frpc的动作，如重新加载；
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_passwd = admin

pool_count = 5              # 初始连接池的数量，默认为0；

tcp_mux = true              # 是否启用tcp复用，默认为true；

user = your_name            # frpc的用户名，用于区别不用frpc的代理；

login_fail_exit = true      # 首次登录失败时退出程序，否则连续重新登录到frps；

protocol = tcp              # 用于连接服务器的协议，支持tcp、kcp、websocket;

dns_server = 8.8.8.8        # 为frp 客户端指定一个单独的DNS服务器；

# start = ssh,dns           # 要启用的代理的名字，默认为空表示所有代理；

# 心跳检查
# heartbeat_interval = 30   # 失败重试次数
# heartbeat_timeout = 90    # 超时时间
~~~



## 五、注册系统服务

~~~shell
[root@server ~]# vim /usr/lib/systemd/system/frps.service
[Unit]
Description=frp server
After=network.target

[Service]
Type=simple

ExecStart=/usr/local/frp_server/frps -c /usr/local/frp_server/frps.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
[root@server ~]# systemctl enable frps
[root@server ~]# systemctl start  frps.service
[root@server ~]# systemctl status frps
~~~

