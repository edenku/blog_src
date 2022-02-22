---
title: SpringBoot同时支持Jsp和Html
categories: SpringBoot
tags: SpringBoot,Spring
date: 2021-11-10
---

## SpringBoot同时支持Jsp和Html

目录结构如下：

![image-20211110173335695](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211110173335695.png)

pom依赖

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!-- springboot对jsp的支持 -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
```

支持多视图解析

```Java
public class HandleResourceViewExists extends InternalResourceView {
    @Override
    public boolean checkResource(Locale locale) {
        File file = new File(this.getServletContext().getRealPath("/") + getUrl());
        //判断页面是否存在
        return file.exists();
    }
}

@Configuration
public class WebConfigure extends WebMvcConfigurationSupport {

    @Bean
    public InternalResourceViewResolver htmlViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/html/");
        //设置检查器
        viewResolver.setViewClass(HandleResourceViewExists.class);
        viewResolver.setSuffix(".html");
        viewResolver.setOrder(0);
        viewResolver.setContentType("text/html;charset=UTF-8");
        return viewResolver;
    }

    @Bean
    public InternalResourceViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        //设置检查器
        viewResolver.setViewClass(HandleResourceViewExists.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        viewResolver.setOrder(0);
        viewResolver.setContentType("text/html;charset=UTF-8");
        return viewResolver;
    }
}
```

测试代码

```java
    @RequestMapping("/test")
    public String test(Model model) {
        model.addAttribute("msg", "hello Jsp");
        return "test";
    }

    @RequestMapping("/html")
    public String test2(Model model) {
        model.addAttribute("msg", "hello China");
        return "html/login";
    }
## JSp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>${msg}</h3>
</body>
</html>

## Html
<html>
<head>
  <meta charset="utf-8">
  <title>登录</title>
</head>

<body>
	this is html page
</body>
</html>
```

测试效果

![image-20211111142636887](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211111142636887.png)