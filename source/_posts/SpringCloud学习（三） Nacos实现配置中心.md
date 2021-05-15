---
title: SpringCloud学习 | 第三篇 SpringCloud整合Nacos实现配置中心
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-14
---

> 接上述文章，我们在auth模块中使用Nacos实现配置中心。

### 一段废话

相比Eureka要专门搭建一个config服务实现配置中心，Nacos集成了配置中心。个人觉得还是值得推荐。

### SpringCloud实现配置中心

#### 添加pom依赖

相比上一文，我们追加了一个spring-cloud-starter-alibaba-nacos-config依赖。

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

        <!-- nacos-config 配置中心依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
```

<!-- more -->

#### 配置文件添加nacos配置

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
  hello-word: Hello Ruipin		 # 配置测试信息
```

#### 添加测试接口读取配置

```java
@RefreshScope
@RestController
@RequestMapping("/sample")
public class SampleController {

    @Value("${example.hello-word}")
    private String word;

    @GetMapping("/hello")
    public Result getWord(){
        return Result.success(this.word);//Result为自定义响应，可以直接返回this.word字符串
    }
}
```

#### 测试

启动服务，第一次调用 http://localhost:8000/sample/hello

![image-20210514161730557](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514161730557.png)

#### 修改配置

打开Nacos界面，配置列表，添加配置：

![image-20210514162229678](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514162229678.png)

Data ID: 输入 ${prefix}-${spring.profiles.active}.${file-extension}，根据我上面的配置，我的配置截图

![image-20210514162723996](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514162723996.png)

点击发布，列表中出现一条配置。

![image-20210514162808803](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514162808803.png)

#### 再次测试

第二次调用 http://localhost:8000/sample/hello，成功实现配置信息的动态刷新。

![image-20210514162839536](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210514162839536.png)



### 总结

到此，SpringCloud整合Nacos实现配置中心就完成了。相比Eureka确实方便了很多。