---
title: Linux安装教程之JDK的安装
date: 2021-05-25
categories: 软件安装
tags: Linux
---

### 下载安装包

[[华为JDK镜像地址](https://repo.huaweicloud.com/java/jdk/)]

```shell
[root@ecs-a7d9 home]# wget https://repo.huaweicloud.com/java/jdk/8u192-b12/jdk-8u192-linux-x64.tar.gz
```

![image-20210525151931568](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210525151931568.png)

<!-- more -->

### 安装

安装包移动至/usr/local 目录。

解压并创建软连接。

```
[root@ecs-a7d9 local]# mv jdk-8u192-linux-x64.tar.gz /usr/local/
[root@ecs-a7d9 local]# cd /usr/local/
[root@ecs-a7d9 local]# tar zxvf jdk-8u192-linux-x64.tar.gz 
[root@ecs-a7d9 local]# ln -s /usr/local/jdk1.8.0_192/ /usr/local/jdk8
```

### 设置环境变量

修改 /etc目录下的profile文件，在末尾追加：

```
export JAVA_HOME=/usr/local/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib/dt.JAVA_HOME/lib/tools.jar:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:${PATH}
```

![image-20210525152756788](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210525152756788.png)

### 验证

```
[root@ecs-a7d9 local]# java -version
-bash: java: command not found
## 使环境变量立即生效
[root@ecs-a7d9 local]# source /etc/profile
[root@ecs-a7d9 local]# java -version
java version "1.8.0_192"
Java(TM) SE Runtime Environment (build 1.8.0_192-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.192-b12, mixed mode)
```

![image-20210525152910866](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210525152910866.png)