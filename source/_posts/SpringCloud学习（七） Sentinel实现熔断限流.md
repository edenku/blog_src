---
title: SpringCloud学习 | 第七篇 Sentinel实现熔断与限流
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-26
---

### 简介

Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

<!-- more -->

Sentinel 的主要特性：

![image-20210526175900928](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210526175900928.png)

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

### 启动控制台

> Sentinel 提供一个轻量级的开源控制台，它提供机器发现以及健康情况管理、监控（单机和集群），规则管理和推送的功能。

https://github.com/alibaba/Sentinel/releases直接下载最新jar文件，我下载的[sentinel-dashboard-1.8.1.jar](https://github.com/alibaba/Sentinel/releases/download/1.8.1/sentinel-dashboard-1.8.1.jar)。

或者下载源码，mvn clean package 构建一个fat jar。

命令行输入如下命令，运行Sentinel控制台。

```
java -jar -Dserver.port=8080 sentinel-dashboard-1.8.1.jar
# PowerShell下需要在参数上加单引号
java -jar ‘-Dserver.port=8080’ sentinel-dashboard-1.8.1.jar
```

启动成功后，http://localhost:8080，访问，使用sentinel/sentinel登录。

![image-20210527140431925](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527140431925.png)

登录以后是个大白控制页面。

### SpringCloud 服务整合Sentinel

此处以我们早先的ruipin-demo服务来做整合测试。添加spring-cloud-starter-alibaba-sentinel依赖

```xml
<!-- Sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

添加Sentinel配置

```yaml
spring:
  application:
    name: ruipin-demo
  cloud:
    nacos:
      discovery:
        server-addr: http://localhost:8848
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yaml
    sentinel:
      transport:
        port: 8010 # 应用与sentinel控制台交互的端口
        dashboard: localhost:8030
feign:
  sentinel:
    enabled: true
```

创建一个http请求接口

```

@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/sayHello/{name}")
    public Result sayHello(@PathVariable String name){
        return Result.success(name);
    }

}
```

到此一个简单的整合就结束了。启动服务后，我们调用接口，并刷新Sentinel控制台。

![image-20210527141326503](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527141326503.png)

实时监控就已经可以看到请求了

![image-20210527141427650](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527141427650.png)

### 配置限流

增加流控。

![image-20210527150323961](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527150323961.png)

也可以再流控规则里直接添加。

![image-20210527150425668](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527150425668.png)

再次调用接口，频繁刷新，会出现限流的默认信息

![image-20210527150548550](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527150548550.png)

### 自定义限流信息

实现默认的BlockExceptionHandler接口，重写限流信息。

```java
@Service
public class DemoBlockHandlerException implements BlockExceptionHandler {

    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e) throws Exception {
        httpServletResponse.setHeader("Content-Type","application/json;charset=UTF-8");
        String message = "{\"code\":999,\"msg\":\"访问人数过多\"}";
        httpServletResponse.getWriter().write(message);
    }
}
```

重新执行接口请求，重启后会发现流控规则已经没有了，需要重新配置。

![image-20210527153656869](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210527153656869.png)

### 使用Nacos存储规则

这里我们还是使用demo模块，添加Pom依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

补充配置，增加Nacos数据源配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        port: 8010
        dashboard: localhost:8030
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: ${spring.application.name}-sentinel
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

Nacos中添加配置

![image-20210528175815289](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528175815289.png)

Json内容：

```json
[
    {
        "resource": "/demo/jiang/{username}",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

配置信息说明：

+ resource 资源名称
+ limitApp 来源应用
+ grade 阈值类型 0 表示线程数，1表示QPS
+ count 单机阈值
+ strategy：流控模式 0 表示直接，1表示关联 2表示链路
+ controlBehavior 流控效果 0 快速是吧 1表示warm up 2 表示排队
+ clusterMode 是否集群

发布以后，我们启动Sentinel 和 demo服务，因为我们目前的Sentinel是懒加载，所以需要手动执行请求http://localhost:8050/demo/jiang/abcdsd 然后发现Sentinel的配置已经同步过来了。

![image-20210528180747821](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528180747821.png)

### 参考文档

+ [[https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D](https://github.com/alibaba/Sentinel/wiki/介绍)]
+ [http://www.macrozheng.com/]
+ [http://www.macrozheng.com/#/cloud/sentinel]

