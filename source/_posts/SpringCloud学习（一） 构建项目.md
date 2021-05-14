---
title: SpringCloud学习 | 第一篇 构建项目
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-12
---

> 公司要对一个项目进行重构，借此机会正好尝试从头搭建一个Cloud微服务架构。

## SpringCloud 与 SpringBoot的版本

因为SpringCloud与SpringBoot的版本需要对应，否则会出现异常。

https://spring.io/projects/spring-cloud#learn 查看目前的版本

![image-20210512141013629](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210512141013629.png)

点击Reference Doc，可以查看该版本对应的SpringBoot版本。

<!-- more -->

## 构建父工程

Idea创建一个Maven项目，选择quickstart，输入你的groupId和artifactId创建一个父工程。

![image-20210512141348116](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210512141348116.png)

父工程仅保留pom文件即可，所以我们删除src目录并修改pom文件，添加版本依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.ruipin</groupId>
	<artifactId>ruipin-cloud</artifactId>
	<version>1.0</version>
	<packaging>pom</packaging>
	<name>ruipin-cloud</name>
	<description>睿聘Cloud</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.5</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<java.version>1.8</java.version>
		<ruipin.version>1.0</ruipin.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<spring-cloud.version>2020.0.2</spring-cloud.version>
		<spring-cloud-alibaba.version>2020.0.RC1</spring-cloud-alibaba.version>
		<lombok.version>1.18.18</lombok.version>
		<hutool.version>5.5.8</hutool.version>
		<mysql.version>8.0.19</mysql.version>
		<druid.version>1.2.4</druid.version>
		<mybatis-plus.version>3.4.2</mybatis-plus.version>
		<minio.version>7.1.0</minio.version>
		<knife4j.version>2.0.8</knife4j.version>
		<redisson.version>3.15.1</redisson.version>
		<swagger.version>1.6.2</swagger.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>${lombok.version}</version>
		</dependency>
		<dependency>
			<groupId>cn.hutool</groupId>
			<artifactId>hutool-all</artifactId>
			<version>${hutool.version}</version>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<!--Spring Cloud 相关依赖-->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!--Spring Cloud & Alibaba 相关依赖-->
			<dependency>
				<groupId>com.alibaba.cloud</groupId>
				<artifactId>spring-cloud-alibaba-dependencies</artifactId>
				<version>${spring-cloud-alibaba.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<dependency>
				<groupId>io.minio</groupId>
				<artifactId>minio</artifactId>
				<version>${minio.version}</version>
			</dependency>

			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>${mysql.version}</version>
			</dependency>

			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>druid-spring-boot-starter</artifactId>
				<version>${druid.version}</version>
			</dependency>

			<dependency>
				<groupId>com.baomidou</groupId>
				<artifactId>mybatis-plus-boot-starter</artifactId>
				<version>${mybatis-plus.version}</version>
			</dependency>

			<dependency>
				<groupId>com.github.xiaoymin</groupId>
				<artifactId>knife4j-spring-boot-starter</artifactId>
				<version>${knife4j.version}</version>
			</dependency>

			<dependency>
				<groupId>com.github.xiaoymin</groupId>
				<artifactId>knife4j-micro-spring-boot-starter</artifactId>
				<version>${knife4j.version}</version>
			</dependency>

			<dependency>
				<groupId>com.github.xiaoymin</groupId>
				<artifactId>knife4j-spring-ui</artifactId>
				<version>${knife4j.version}</version>
			</dependency>

			<!-- 分布式锁 -->
			<dependency>
				<groupId>org.redisson</groupId>
				<artifactId>redisson</artifactId>
				<version>${redisson.version}</version>
			</dependency>


			<dependency>
				<groupId>io.swagger</groupId>
				<artifactId>swagger-annotations</artifactId>
				<version>${swagger.version}</version>
			</dependency>

		</dependencies>
	</dependencyManagement>

</project>

```

到此，一个父工程就创建完毕了。

![image-20210512141721341](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210512141721341.png)



### 总结

接下来我们开始搭建Nacos服务。

