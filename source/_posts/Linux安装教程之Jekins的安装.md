---
title: Linux安装教程之Jekins的安装
date: 2021-07-02
categories: 软件安装
tags: Linux

---

## 环境

  ```
# 当前操作系统版本
[root@localhost ~]# cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core) 
# 查看JDK版本
[root@localhost ~]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
# 查看JDK安装目录
[root@localhost ~]# echo $JAVA_HOME$
/usr/local/jdk1.8.0_181$
  ```

> 如果没有安装JDK 请先安装JDK

<!--more-->

## 软件安装

### 安装 Java

> 如已安装，请跳过此节

  + 下载安装包，我选择的JDK8[官网地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

  + 上传到服务器`/usr/local`目录。

  + 解压并创建软连接。

  ```
  [root@localhost local]# tar zxvf jdk-8u181-linux-x64.tar.gz 
  [root@localhost local]# ln -s /usr/local/jdk1.8.0_181/ /usr/local/jdk8
  ```

  + 设置环境变量 `vi /etc/profile`

  ```
  export JAVA_HOME=/usr/local/jdk8
  export JRE_HOME=${JAVA_HOME}/jre
  export CLASSPATH=.:${JAVA_HOME}/lib/dt.JAVA_HOME/lib/tools.jar:${JRE_HOME}/lib
  export PATH=${JAVA_HOME}/bin:${PATH}
  ```

  + `source /etc/profile` 使环境变量立即生效

  + `java -version` 验证

  ```
  [root@localhost local]# source /etc/profile
  [root@localhost local]# java -version
  java version "1.8.0_181"
  Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
  Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
  ```

### 安装 Git

  + 安装编译源码的工具

  `yum -y groupinstall "Development Tools"`

  + 下载依赖包

  `yum -y install zlib-devel perl-ExtUtils-MakeMaker asciidoc xmlto openssl-devel`

  + 下载git 源码 并解压

  ```
  [root@localhost ~]# wget https://github.com/git/git/archive/v2.19.0.tar.gz
  [root@localhost ~]# mv v2.19.0.tar.gz /usr/local/
  [root@localhost ~]# tar -zxvf v2.19.0.tar.gz
  ```

  `wget https://github.com/git/git/archive/v2.19.0.tar.gz`

  **登录https://github.com/git/git/releases查看git的最新版。不要下载带有-rc的，因为它代表了一个候选发布版本**

  + 编译

  ```
  [root@localhost git-2.19.0]# cd git-2.19.0/
  [root@localhost git-2.19.0]# make configure
  [root@localhost git-2.19.0]# ./configure --prefix=/usr/local/git
  [root@localhost git-2.19.0]# make && make install
  [root@localhost git-2.19.0]# export PATH="/usr/local/git/bin:$PATH" 
  [root@localhost git-2.19.0]# source /etc/profile
  [root@localhost git-2.19.0]# git --version 
   
  ```

### 安装 maven

  + 下载maven[地址](http://maven.apache.org/download.cgi)

  + 解压至 `/usr/local`

  + 设置环境变量并使其立即生效

  + `mvn -v` 验证

  ```
  # MAVEN
  export MAVEN_HOME=/usr/local/apache-maven-3.5.4
  export MAVEN_HOME
  export PATH=$PATH:$MAVEN_HOME/bin
  ```

### 安装 Tomcat

  + 下载安装包，选择9.0.12版本

  ```
  [root@localhost ~]# wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.12/bin/apache-tomcat-9.0.12.tar.gz
  --2018-09-26 10:38:25--  http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.12/bin/apache-tomcat-9.0.12.tar.gz
  Resolving mirrors.hust.edu.cn (mirrors.hust.edu.cn)... 202.114.18.160
  Connecting to mirrors.hust.edu.cn (mirrors.hust.edu.cn)|202.114.18.160|:80... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 9912675 (9.5M) [application/octet-stream]
  Saving to: ‘apache-tomcat-9.0.12.tar.gz’
  ```

  + 解压并移至`/usr/local`目录。

  ```
  [root@localhost ~]# tar -xzvf apache-tomcat-9.0.12.tar.gz 
  [root@localhost ~]# mv apache-tomcat-9.0.12 /usr/local/apache-tomcat-9.0.12
  ```

  + 创建软连接 

  ```
  # 软连接类似Windows的快捷方式
  [root@localhost ~]# ln -s /usr/local/apache-tomcat-9.0.12 /usr/local/tomcat
  ```

  + 修改默认Tomcat端口

  ```
  [root@localhost ~]# vi /usr/local/tomcat/conf/server.xml
  ...
  <Connector port="10000" protocol="HTTP/1.1"
     connectionTimeout="20000" redirectPort="8443" />
  ...
  ```

  默认端口8080，修改为10000

  + 启动

  `/usr/local/tomcat/bin/startup.sh` 

  使用 http://IP:PORT 访问。    

  > 如果启动正常，但无法正常访问。(ping可以，端口telnet不通)
  > 需要关闭防火墙或开放指定端口

  **关闭防火墙 (CentOS 7)**

  + 启动： systemctl start firewalld
  + 重启： systemctl restart firewalld.service
  + 关闭： systemctl stop firewalld
  + 查看状态： systemctl status firewalld 
  + 开机禁用  ： systemctl disable firewalld
  + 开机启用  ： systemctl enable firewalld

  **开放指定端口**

  ```
  ## --zone=public：表示作用域为公共的；
  ## --add-port=8080/tcp：添加tcp协议的端口8080；
  ## --permanent：永久生效，如果没有此参数，则只能维持当前服务生命周期内，重新启动后失效；
  [root@localhost local]# firewall-cmd --zone=public --add-port=10380/tcp --permanent;
  success
  ## 重启防火墙
  [root@localhost local]# systemctl restart firewalld.service
  [root@localhost local]# 

  ```

### 安装 Jenkins

  + 下载war包[地址](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)

  + 上传war包到`/usr/local/tomcat/webapps/`目录。

  + 重启tomcat.

  + `http://IP:PORT/jenkins/` 访问jenkins。

  + `/root/.jenkins/secrets/initialAdminPassword` 复制密码 解锁Jenkins。

  + 安装社区推荐的插件。

  **设置管理员密码**

  admin/admin

  > 设置完密码以后，使用设置的管理员密码登录，访问jenkins，白页面。猜测可能与端口设置有关，
  > 修改端口为10380后，重启tomcat，可以正常访问。

## 小结

  到此，我们的Jenkins环境就安装完成，后面我们将安装部分插件并使用git+maven构建任务。

## 参考资料

  + [Linux下安装Java（JDK8）](https://www.cnblogs.com/liugh/p/6623530.html)
  + [CentOS7安装最新版git教程](https://blog.csdn.net/Juladoe/article/details/76170193)
  + [CentOS7使用firewalld打开关闭防火墙与端口](https://www.cnblogs.com/moxiaoan/p/5683743.html)  
  + [Linux安装Tomcat](https://www.cnblogs.com/yuhebin/p/8594774.html)  
  + [linux下安装及配置jenkins](https://blog.csdn.net/andyzhaojianhui/article/details/73472500)  



