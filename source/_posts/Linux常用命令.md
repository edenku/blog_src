---
title: Linux常用命令
date: 2021-05-14
cateogries: 常用命令
---

###  防火墙

**查看防火墙状态**

systemctl status firewalld.service

firewall-cmd --state

**关闭防火墙**

systemctl stop firewalld.service

**开启防火墙**

systemctl start firewalld.service

**禁用防火墙**

systemctl disable firewalld.service

<!-- more -->

**设置开机自动启动**

systemctl enable firewalld.service

 **关闭开机启动**

systemctl disable firewalld.service

**查看防火墙所有开放的端口**

firewall-cmd --zone=public --list-ports

### 查看系统版本

cat /etc/redhat-release