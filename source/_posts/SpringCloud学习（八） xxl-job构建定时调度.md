---
title: SpringCloud学习 | 第八篇 xxl-job构建定时调度
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-31
---

> 官网地址：[[https://www.xuxueli.com/xxl-job/](https://www.xuxueli.com/xxl-job/)]
>
> XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

### 下载

[中文文档]([https://www.xuxueli.com/xxl-job/](https://www.xuxueli.com/xxl-job/))

源码仓库地址：

| 仓库地址                             | Release Download                                          |
| ------------------------------------ | --------------------------------------------------------- |
| https://github.com/xuxueli/xxl-job   | [Download](https://github.com/xuxueli/xxl-job/releases)   |
| http://gitee.com/xuxueli0323/xxl-job | [Download](http://gitee.com/xuxueli0323/xxl-job/releases) |

Maven依赖Pom

```xml
<!-- http://repo1.maven.org/maven2/com/xuxueli/xxl-job-core/ -->
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>${最新稳定版本}</version>
</dependency>
```

### 开始集成

#### 初始化数据库

我下的2.3.0版本，执行数据库初始化SQL脚本

脚本位置：xuxueli0323-xxl-job-2.3.0\xxl-job\doc\db\tables_xxl_job.sql 

#### 导入调度中心与公共依赖模块

新建一个xxl-job模块，导入下载的源码。

```
xxl-job-admin：调度中心
xxl-job-core：公共依赖
xxl-job-executor-samples：执行器Sample示例
```

修改properties配置文件，使用自己的数据库配置，然后启动admin服务，使用账号admin / 123456登录，

![image-20210603172138134](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210603172138134.png)

#### 构建自己的Job模块

新建ruipin-job模块

添加Pom依赖

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
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.3.0</version>
        </dependency>
    </dependencies>
```

添加配置文件

```yaml
server:
  port: 8120
spring:
  application:
    name: ruipin-job
  cloud:
    nacos:
      discovery:
        server-addr: http://127.0.0.1:8848
      config:
        file-extension: yaml      # 默认值 properties
        group: DEFAULT_GROUP      # 默认值 DEFAULT_GROUP
        prefix: ${spring.application.name}
xxl:
  job:
    admin:
      addresses: http://127.0.0.1:8080/xxl-job-admin
    executor:
      appname: ruipin-job # 这个需要在xxl_job_group中添加记录
      address:
      port: 8130
      logpath: /data/applogs/xxl-job/jobhandler
      logretentiondays: 30
feign:
  okhttp:
    enabled: true

```

添加一个任务组 

```sql
INSERT INTO `xxl_job`.`xxl_job_group`(`id`, `app_name`, `title`, `address_type`, `address_list`, `update_time`) VALUES (2, 'ruipin-job', '睿聘执行器', 0, NULL, '2021-06-04 10:51:14');

```

代码编写

XxlJobConfig是demo中的配置

```java
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken:}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip:}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;


    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }
}
```

自己的一个定时任务

```java
@Component
@Slf4j
public class SampleXxlJob {

    @Autowired
    private DemoFeignClient demoFeignClient;

    /**
     * 1、简单任务示例（Bean模式）
     */
    @XxlJob("ruipinDemoJob")
    public void demoJobHandler() throws Exception {
        XxlJobHelper.log("XXL-JOB, Hello World.");
        Result<String> ruipin = demoFeignClient.sayHello("ruipin");
        log.info("调用结果： " + ruipin);
        XxlJobHelper.log("CCCCCCCCCCCCCCC" + ruipin.getCode() + " : " + ruipin.getMsg());
    }
}
```

启动类

```
@SpringBootApplication(scanBasePackages = {"com.ruipin.app.*", "com.ruipin.job.*"})
@EnableDiscoveryClient
@EnableFeignClients("com.ruipin.app.*")
public class JobApplication {
    public static void main(String[] args) {
        SpringApplication.run(JobApplication.class, args);
    }

}
```

#### 配置

![image-20210604154804976](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210604154804976.png)

配置上面几项即可。操作手动执行一次，就可以看到调度日志中的结果了。

![image-20210604154923206](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210604154923206.png)

### 高级用法

TODO...

