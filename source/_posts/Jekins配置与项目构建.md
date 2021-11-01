---
title: Jekins配置与项目构建
date: 2021-08-12
categories: 软件安装
tags: Windows, jekins

---

### 配置Git

![image-20210812144931741](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812144931741.png)

## 配置JDK

![image-20210812145310634](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812145310634.png)

## 配置maven

![image-20210812145516379](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812145516379.png)

到这里，我们就已经可以正常构建了。

![image-20210812150551294](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812150551294.png)

### 配置自动构建job

新建一个maven任务

![image-20211028174212266](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211028174212266.png)

添加描述配置历史构建策略

![image-20211028174409187](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211028174409187.png)

添加代码地址

![image-20211028174820405](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211028174820405.png)

指定分支

![image-20211029145400866](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029145400866.png)

这里也可以将分支参数化。后续说明

指定pom并执行打包命令

> clean package -Ptest -Dmaven.test.skip=true # -Ptest是我的配置文件差异，默认可以不要

![image-20211029145507909](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029145507909.png)

Post Steps

打包完成后，执行Shell脚本进行本地部署

![image-20211029145713737](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029145713737.png)

```shell
## 备份
rm -rf /home/bak/pro-xxx.war
cp -rf /home/tomcat/webapps/pro-xxx.war /home/bak/
## 拷贝
mv -f target/pro-xxx.war /home/tomcat/webapps/
## kill 进程
ps -ef|grep tomcat/ |grep -v grep | awk '{print $2}' | xargs -r kill -9
## 重启
/home/tomcat/bin/startup.sh
```

到这儿，一个简单的Java-Maven项目的构建job就配置完了。

> 这里部署的时候遇到一个问题，console的日志里看到tomcat正常启动，但是却看不到tomcat进程。查询原因是Jenkins默认会在build后kill掉所有的衍生进程。
>
> 解决方案1：
>
> 1. execute shell中加入：BUILD_ID=DONTKILLME,即可防止jenkins杀死启动的tomcat进程
> 2. 修改/etc/sysconfig/jenkins配置，在JENKINS_JAVA_OPTIONS中加入-Dhudson.util.ProcessTree.disable=true 并重启jenkins。