---
title: SpringCloud学习 | 第二篇 SpringCloud整合Nacos实现注册中心
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-14
---

> 接上述文章，我们开始搭建Nacos服务。

### 前言

**为什么要使用Nacos，不使用Eureka呢**

1. Eureka只实现注册中心，nacos同时实现注册中心与配置中心。
2. Eureka2.0 闭源, Nacos国产各种中文文档。

### 搭建Nacos服务

#### 下载Nacos-server

[GitHub地址](https://github.com/alibaba/nacos/releases) 。目前nacos-server的最新版本为2.0.1，不同版本可能些许差异。

![image-20210514105212761](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514105212761.png)

下载到本地目录并解压，我放在了我代码同级目录。

![image-20210514105651639](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514105651639.png)

<!-- more -->

#### 创建数据库

使用nacos\conf\nacos-mysql.sql脚本创建好数据库表和数据初始化。库名：nacos_config

#### 配置数据库连接

编辑nacos\conf/application.properties，修改自己本地的数据库连接信息	

![image-20210514110529653](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514110529653.png)

#### 启动nacos-server服务

☆ 确保本机已配置JAVA环境变量，版本8+

nacos\bin\下执行 startup.cmd -m standalone 启动。-m standalone代表单机模式。默认以集群模式启动。

![image-20210514112333044](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514112333044.png)

#### 进入nacos管理控制台

启动服务后，登录控制台看看吧。地址: http://localhost:8848/nacos. 账号/密码: nacos/nacos

![image-20210514112527620](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514112527620.png)

### SpringCloud整合Nacos

#### 版本说明

Nacos Server：2.0.1

SpringBoot：   2.4.5

SpringCloud： 2020.0.2

SpringCloud Alibaba：2.2.1.RELEASE

#### 服务注册到Nacos

新建一个module，命名为ruipin-auth

**POM依赖**

```xml
<dependencies>
     	<!-- SpringCloud从2020版本后对配置文件加载进行了重构，
			默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
```

**启动入口开启服务发现**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```

**访问Nacos服务列表**查看

![image-20210514145800211](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514145800211.png)



### 总结

到此SpringCloud整合Nacos注册中心就完成了，后面我们继续使用Nacos完成配置中心配置。