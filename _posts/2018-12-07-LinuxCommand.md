---
title: 常用命令
date: 2018-12-07 12:08:12
categories:
- linux
tags:
- linux
- command
---

# 常用命令

## scp
``` shell
# 拷贝远程文件到本地
scp remote_username@remote_ip:remote_folder/file_name .
scp remote_username@remote_ip:remote_folder/file_name local_folder
scp -r remote_username@remote_ip:remote_folder local_folder

# 拷贝本地文件到远程
scp local_file remote_username@remote_ip:remote_folder
scp -r local_folder remote_username@remote_ip:remote_folder
```
<!--more-->

## tar
``` shell
# 压缩
tar -czvf code_bk.tar.gz code/

# 解压
tar -xzvf code_bk.tar.gz
```

## locate
``` shell
# 更新 /var/lib/slocate/slocate.db 数据库
locate -u

locate file_name
```

## which
``` shell
# 在环境变量$PATH中查找符合条件的文件
which file_name
```

## host
``` shell
host hostname
```

## ps
``` shell
# 显示所有包含其他使用者的进程（USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND）
ps aux 
```


## grep
```shell
# 把匹配的行打印出来，grep全称是Global Regular Expression Print
sshd -T | grep -i max
```
-i 或 --ignore-case : 忽略字符大小写的差别。
-n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的编号


## cut
```shell
cut -d: -f1 /etc/passwd
```
分割符 ':'   ，第一列


## adduser
```shell
# 添加用户
$sudo adduser dev           
```


## usermod
```shell
# 用户添加sudo权限
$sudo usermod -aG sudo dev 
```
