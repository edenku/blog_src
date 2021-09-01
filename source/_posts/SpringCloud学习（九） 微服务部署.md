---
title: SpringCloud学习 | 第九篇 微服务部署
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-06-21
---

### Jekins安装

#### Windows下的安装

#### Linux下的安装



### 打包

#### Pom依赖

```
<build>
  <plugins>
    <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
       <configuration>
         <executable>true</executable>
       </configuration>
    </plugin>
  </plugins>
</build>
```

mvn clean package -Dskip.test=true 构建fat-jar

### 启动

