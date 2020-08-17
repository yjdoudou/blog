---
title: docker安装frp内网穿透
tags:
  - 内网穿透
  - docker
categories: 部署
abbrlink: 57486
date: 2020-08-05 09:35:40
---

# 服务端

## 创建frp配置文件

```shell
# 创建frp文件夹
mkdir -p frp && cd frp
# 创建frps.ini
cat <<EOF> frps.ini
# 复制如下配置，自行修改密码
[common]
bind_port = 10000
vhost_http_port = 10001
vhost_https_port = 10002
dashboard_addr = 0.0.0.0
dashboard_port = 10003
dashboard_user = zyj
dashboard_pwd = xxx
EOF
```

<!--more-->

## 启动docker容器

```shell
# 创建启动脚本
cat <<EOF> start.sh
# 复制如下配置，挂载容器的frps.ini目录请自行修改
#!/bin/bash
docker run -d \\
    --restart always \\
    --network host \\
    --name frps \\
    -v /usr/local/project/frp/frps.ini:/etc/frp/frps.ini \\
    snowdreamtech/frps
EOF
# 运行脚本
sh start.sh
```

访问``公网ip:10003``，输入账号密码，看到frp管理界面。

# 客户端

## win10

下载客户端<https://github.com/fatedier/frp/releases/download/v0.30.0/frp_0.30.0_windows_amd64.zip>

解压，修改`frpc.ini`文件

```shell
# 公网ip
server_addr = 255.255.255.255
server_port = 10000

[portal]
type = http
# 本地服务端口
local_port = 8081
# 公网ip或者域名
custom_domains = 255.255.255.255
```

## linux

1. 创建frp配置文件,remote_port记得在公网放开防火墙

```shell
# 创建frp文件夹
mkdir -p frp && cd frp
# 创建frps.ini
cat <<EOF> frps.ini
# 复制如下配置
[common]
server_addr = 公网ip
server_port = 10000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[nginx]
type = tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 6001
EOF
```

2. 启动docker容器

```shell
# 创建启动脚本
cat <<EOF> start.sh
# 复制如下配置，挂载容器的frps.ini目录请自行修改
#!/bin/bash
docker run -d \\
    --restart always \\
    --network host \\
    --name frpc \\
    -v /usr/project/frp/frpc.ini:/etc/frp/frpc.ini \\
    snowdreamtech/frpc
EOF
# 运行脚本
sh start.sh
```