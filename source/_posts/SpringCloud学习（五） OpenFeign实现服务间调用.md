---
title: SpringCloud学习 | 第五篇 SpringCloud整合OpenFeign实现服务间调用
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-15
---

### SpringCloud实现服务间调用

本文涉及模块

| 模块            | 端口 | 描述                    |
| --------------- | ---- | ----------------------- |
| Nacos server    | 8848 | 注册中心与配置中心      |
| ruipin-auth     | 8000 | 服务调用方              |
| ruipin-demo     | 8050 | 服务提供方              |
| ruipin-demo-api | -    | api模块，提供给auth引用 |

#### 服务提供方

新建模块ruipin-application模块，管理所有业务功能。在application中建立ruipin-demo，提供服务。

<!-- more -->

**POM依赖**

```xml
<dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- SpringCloud从2020版本后对配置文件加载进行了重构，
    默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>

    <!-- 注册中心 -->
    <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>

    <!-- 配置中心 -->
    <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>

    <dependency>
    <groupId>com.ruipin</groupId>
    <artifactId>ruipin-demo-api</artifactId>
    <version>${ruipin.version}</version>
    </dependency>
</dependencies>
```

**配置文件**

```yaml
server:
  port: 8050

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

```

**服务提供**

一个DemoController类，提供一个接口，返回信息。

一个启动入口类。

```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/sayHello/{name}")
    public Result sayHello(@PathVariable String name){
        return Result.success(name);
    }

}

@SpringBootApplication
@EnableDiscoveryClient
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### 服务的API

在application模块中，建立子模块，ruipin-api-demo。提供feigin远程调用

**POM依赖**

```xml
<dependencies>

    <!-- 远程调用HTTP客户端 -->
    <!-- openfeign依赖 1. http客户端选择okhttp 2. loadbalancer替换ribbon -->
    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>

    <dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    </dependency>

    <dependency>
    <groupId>com.ruipin</groupId>
    <artifactId>common-core</artifactId>
    <version>${ruipin.version}</version>
    </dependency>
</dependencies>
```

**Feign调用**

创建Feign调用接口并提供一个降级回调服务。

```java
@FeignClient(value = "ruipin-demo", fallback = DemoFeignFallback.class)
public interface DemoFeignClient {

    @GetMapping("/demo/sayHello/{name}")
    Result<String> sayHello(@PathVariable String name);
}
@Component
@Slf4j
public class DemoFeignFallback implements DemoFeignClient {

    @Override
    public Result<String> sayHello(String name) {
        log.error("feign远程调用系统用户服务异常后的降级方法");
        return Result.failed(ResultCode.DEGRADATION);
    }
}
```

#### 服务调用方

ruipin-auth模块 基于之前的依赖，添加ruipin-demo-api依赖，全部依赖如下：

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
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
        <!-- 注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!-- nacos-config 配置中心依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>


        <dependency>
            <groupId>com.ruipin</groupId>
            <artifactId>common-core</artifactId>
            <version>${ruipin.version}</version>
        </dependency>

        <dependency>
            <groupId>com.ruipin</groupId>
            <artifactId>ruipin-demo-api</artifactId>
            <version>${ruipin.version}</version>
        </dependency>
    </dependencies>
```

**配置文件**

配置文件将feign底层调用修改为okhttp。全部配置内容有

```yaml
server:
  port: 8000
spring:
  application:
    name: ruipin-auth
  cloud:
    nacos:
      discovery:
        server-addr: http://127.0.0.1:8848
      config:
        file-extension: yaml      # 默认值 properties
        group: DEFAULT_GROUP      # 默认值 DEFAULT_GROUP
        prefix: ${spring.application.name}
example:
  hello-word: Hello Ruipin
feign:
  okhttp:
    enabled: true
```

**服务调用代码**

```java
@SpringBootApplication
@EnableDiscoveryClient
// 开启feign调用
@EnableFeignClients
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}

@RefreshScope
@RestController
@RequestMapping("/sample")
public class SampleController {

    @Autowired
    private DemoFeignClient demoFeignClient;
	//调用ruipin-demo的服务
    @GetMapping("/sayHello/{name}")
    public Result sayHello(@PathVariable String name){
        return demoFeignClient.sayHello(name);
    }


}
```

#### 测试

到此，我们的服务调用代码就整合完毕，依次启动nacos server、ruipin-auth、ruipin-demo三个模块。

打开localhost:8848/nacos，可以看到ruipin-auth和ruipin-demo服务已经启动

![image-20210515114749158](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210515114749158.png)

我们先直接调用demo的接口服务，调用正常。

![image-20210515114656194](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210515114656194.png)

然后我们调用auth服务，也可以正常调用。

![image-20210515114902468](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210515114902468.png)

#### 总结

到此，我们的服务间调用就整合完毕了。