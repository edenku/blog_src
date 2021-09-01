---
title: Windows安装教程之Jekins的安装
date: 2021-07-30
categories: 软件安装
tags: Windows, jekins

---

## 安装

### 系统配置

OS: Windows 10

JDK: 1.8.0_171

Maven: 3.6.3

### 安装步骤

#### 下载

[官网](https://www.jenkins.io/) 下载Jenkins安装包。

![image-20210730104534113](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210730104534113.png)

#### 安装

一路傻瓜安装即可。

#### 启动

这里直接使用命令启动。管理员权限打开cmd或Power Shell。

```
java -jar jenkins.war --httpPort=8080
```

启动Jenkins服务。看到如下信息表示启动成功。

![image-20210730104830318](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210730104830318.png)

## 配置



