---
title: Maven常用命令
date: 2021-06-03
tags: Maven
---

### 如何添加本地jar包到maven仓库

安装指定文件到本地仓库命令：mvn install:install-file

**cmd打开命令行，不要用powerShell**

```
-DgroupId=<groupId>       : 设置项目代码的包名(一般用组织名)
-DartifactId=<artifactId> : 设置项目名或模块名 
-Dversion=1.0.0           : 版本号
-Dpackaging=jar           : 什么类型的文件(jar包)
-Dfile=<myfile.jar>       : 指定jar文件路径与文件名(同目录只需文件名)
安装命令实例：
mvn install:install-file -DgroupId=io.netty -DartifactId=netty-all -Dversion=4.1.58.Final -Dpackaging=jar -Dfile=D:\netty-all-4.1.58.Final.jar
```

![image-20210603164713209](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210603164713209.png)