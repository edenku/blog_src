---
title: SpringCloud学习 | 第四篇 SpringCloud整合Gateway实现API网关
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-14
---

> 接下来继续配置Gateway。

### 前言

**微服务网关的作用有哪些？**

1. 鉴权。
2. 统一入口，保障服务安全性。
3. 限流熔断统一配置。
4. 路由转发。
5. 负载均衡
6. 链路跟踪。

**为什么使用Gateway实现API网关？**

SpringCloud目前常用的网关有Netflix开源的Zuul和Spring自己的Gateway。

<!-- more -->

### SpringCloud实现API网关

本文涉及的微服务模块有：

| 模块           | 端口 | 描述           |
| -------------- | ---- | -------------- |
| Nacos Server   | 8848 | 注册与配置中心 |
| ruipin-gateway | 9999 | 网关           |
| ruipin-auth    | 8000 | 认证及服务demo |

#### 新建ruipin-gateway模块

#### 添加POM依赖

```xml
<dependencies>
        <!-- SpringCloud从2020版本后对配置文件加载进行了重构，
           默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
		<!-- 2020Cloud版本，不加该依赖可能在路由转发时会报503 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>

        <!-- 注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
    </dependencies>
```

#### 配置文件

配置nacos注册和路由转发相关

```yaml
server:
  port: 9999

spring:
  application:
    name: ruipin-gateway
  cloud:
    nacos:
      # 注册中心
      discovery:
        server-addr: http://127.0.0.1:8848
      # 配置中心
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yaml
    gateway:
      discovery:
        locator:
          enabled: true  # 启用动根据服务id生成路由
          lower-case-service-id: true # 设置路由的路径为小写的服务ID
      routes:
        - id: ruipin-auth
          uri: lb://ruipin-auth
          predicates:
            - Path=/auth/**  # 所有/auth/**开头的请求都会转发到 lb://ruipin-auth
          filters:
            - StripPrefix=1 # 移除前缀 /auth
```

#### 启动入口

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

#### 测试

依次启动Nacos-server、ruipin-auth、ruipin-gateway。

![image-20210514181859300](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514181859300.png)

服务调用9999端口网关服务，/auth/开头的请求转发到了 ruipin-auth下的/sample/hello请求，测试成功。

### 总结

到此，一个简单的微服务框架就快要成型了。接下来我们会继续整合OAuth2进行网关鉴权等。

### 参考连接

+ [Spring Cloud实战 | 第四篇：Spring Cloud整合Gateway实现API网关](https://www.cnblogs.com/haoxianrui/p/13608650.html)

+ [SpringCloud 2020版本教程2：使用spring cloud gateway作为服务网关](https://www.fangzhipeng.com/springcloud/2021/04/03/sc-2020-gateway.html)