---
title: Linux安装教程之Redis的安装
date: 2021-06-21
categories: 软件安装
tags: Linux
---

### 下载安装包

下载地址：https://redis.io/download

当前最新版本6.2.4，下载并按照。

```shell
wget https://download.redis.io/releases/redis-6.2.4.tar.gz
tar xzf redis-6.2.4.tar.gz
cd redis-6.2.4
make
```

编译后，src目录执行即可。

<!-- more -->

```
./bin/redis-server
```

如果需要后台启动，修改redis.conf

![image-20210621155558248](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210621155558248.png)

重新./redis-server redis.conf启动，即可。

ps -ef|grep redis可以查看是否启动成功。

