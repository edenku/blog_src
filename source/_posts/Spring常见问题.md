---
title: Spring常见问题
categories: SpringCloud
tags: SpringCloud
date: 2021-11-05
---

## Spring常见问题

### 父模块引入子模块如何

有一个子模块时数据层模块，配置文件包含数据库jdbc相关级数据源datasource相关配置。父模块直接引入子模块的时候是不会加载子模块的配置文件，需要主动加载。

```java
@Configuration
@PropertySource("classpath:application-dat.properties")
public class DataSourceConfiguration {}
```

### SpringBoot 支持Hibernate

[代码地址: springboot-hql-jsp](https://gitee.com/ruocy/spring-demo/tree/master/)

### SpringBoot 集成Jsp

**添加依赖**

```xml
<!-- springboot对jsp的支持 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

**声明视图解析器**

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
    static-path-pattern: /static/**
```

新建一个jsp目录，沿用以前MVC的方式。建立webapp/WEB-INF/jsp

![image-20211109154747619](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211109154747619.png)

同时在idea中配置web资源目录

![image-20211109154941810](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211109154941810.png)

**idea运行时，如果涉及到maven父子结构的项目时，需要再配置中修改工作目录，修改为$MODULE_WORKING_DIR$**

![image-20211109155210808](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211109155210808.png)

到此就配置完毕了，写一个测试jsp，就可以看看效果了。

```
## controller
@RequestMapping("/test")
public String test(Model model) {
    model.addAttribute("msg", "hello");
    return "test";
}
    
## Jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>${msg}</h3>

```

![image-20211109171105988](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211109171105988.png)

代码地址: [springboot-hql-jsp](https://gitee.com/ruocy/spring-demo/tree/master/)

参考链接：[SpringBoot集成jsp](https://blog.csdn.net/weixin_43823808/article/details/115732826)





