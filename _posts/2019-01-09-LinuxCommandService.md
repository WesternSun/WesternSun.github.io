---
title: 常用命令
date: 2019-01-09 00:44:56
categories:
- linux
tags:
- linux
---

# Linux 服务管理两种方式service和systemctl

1.service命令
service命令其实是去/etc/init.d目录下，去执行相关程序

```shell
# service命令启动redis脚本
service redis start
# 直接启动redis脚本
/etc/init.d/redis start
# 开机自启动
update-rc.d redis defaults
其中脚本需要我们自己编写
```
<!--more-->
2.systemctl命令
Systemd 是 Linux 系统工具，用来启动守护进程，已成为大多数发行版的标准配置。systemctl是 Systemd 的主命令，用于管理系统，是Linux系统最新的初始化系统(init),作用是提高系统的启动速度，尽可能启动较少的进程，尽可能更多进程并发启动。
systemd对应的进程管理命令是systemctl

systemctl命令兼容了service
即systemctl也会去/etc/init.d目录下，查看，执行相关程序

```shell
systemctl redis start
systemctl redis stop
# 开机自启动
systemctl enable redis
```