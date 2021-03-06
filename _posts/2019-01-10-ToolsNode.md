---
title: Node.js
date: 2019-01-10 00:09:51
categories:
- Tools
tags:
- tool
- Node.js
---
# Node.js

Node.js是一个Javascript运行环境(runtime environment)，发布于2009年5月，由Ryan Dahl开发，实质是对Chrome V8引擎进行了封装。Node.js对一些特殊用例进行优化，提供替代的API，使得V8在非浏览器环境下运行得更好。 V8引擎执行Javascript的速度非常快，性能非常好。Node.js是一个基于Chrome JavaScript运行时建立的平台， 用于方便地搭建响应速度快、易于扩展的网络应用。Node.js 使用事件驱动， 非阻塞I/O模型而得以轻量和高效，非常适合在分布式设备上运行数据密集型的实时应用。
<!--more-->

## 环境搭建

ubuntu上apt安装的nodejs版本较旧，通过一下命令进行版本更新
```shell
sudo apt update -y
sudo apt install -y nodejs nodejs-legacy npm
sudo npm config set registry https://registry.npm.taobao.org
sudo npm install n -g
sudo n stable
```

## cnpm

安装cnpm
```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org

# 使用cnpm进行安装，使用方法和npm相同
cnpm install -g electron
```