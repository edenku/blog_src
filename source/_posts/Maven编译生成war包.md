---
title: Maven编译生成war包
date: 2021-08-12
tags: Maven
---

> 接手项目平时的构建都是手动上传编译后的文件，比较麻烦。这里梳理构建war包的过程并记录构建中遇到的各类问题。

## 构建war包

1. mvn clean package 打包报错，“编码GBK的不可映射字符”。

   ![image-20210812101658940](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812101658940.png)

   检查步骤：

   <!-- more -->

   确保idea的设置都是UTF-8编码。

   ![image-20210812102153378](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210812102153378.png)

      确保pom文件中已添加UTF-8的配置

   ```xml
   <properties>		
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <maven.compiler.source>1.8</maven.compiler.source>
       <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

   我这里在添加pom的配置后，就已经解决了。

2. mvn clean package 打包报错，“程序包com.alipay.demo.trade.service.impl不存在”

   这个报错是因为我的maven项目引入了本地的jar包。pom引入本地文件即可。

   ```xml
   <dependency>
       <groupId>com.alipay</groupId>
       <artifactId>alipay-trade-sdk-20161215</artifactId>
       <version>3.0.0</version>
       <scope>system</scope>
       <systemPath>${project.basedir}/src/main/webapp/WEB-INF/lib/alipay-trade-sdk-20161215.jar</systemPath>
   </dependency>
   ```

## 配置多环境文件

找出多环境下配置文件的差异，新建dev、prod等目录。针对不同环境打包不同环境的配置。

添加环境配置与拷贝配置文件的插件

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <package.environment>dev</package.environment>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <package.environment>prod</package.environment>
        </properties>
    </profile>
</profiles>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>compile</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <!-- 覆盖原有文件 -->
                <overwrite>true</overwrite>
                <outputDirectory>${project.build.outputDirectory}</outputDirectory>
                <!-- 也可以用下面这样的方式（指定相对url的方式指定outputDirectory） <outputDirectory>target/classes</outputDirectory> -->
                <!-- 待处理的资源定义 -->
                <resources>
                    <resource>
                        <!-- 指定resources插件处理哪个目录下的资源文件 -->
                        <!--suppress UnresolvedMavenProperty -->
                        <directory>src/main/resources/${package.environment}</directory>
                        <filtering>false</filtering>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这样再执行打包的时候：mvn clean package -Pprod即可将配置文件打包到war包内了。

## 其他

### pom内置属性

比如一个maven项目名为：maven-demo

| 属性                             | 含义              | 示例                                           |
| -------------------------------- | ----------------- | ---------------------------------------------- |
| ${project.basedir}               | 包含pom文件的目录 | maven-demo/                                    |
| ${project.build.sourceDirectory} | 项目主源码目录    | src/main/java                                  |
| ${project.build.outputDirectory} | 构建过程输出目录  | target/classes                                 |
| ${project.build.finalName}       | build结果名称     | 缺省为${project.artifactId}-${project.version} |
| ${project.packaging}             | 打包类型          | 缺省为jar                                      |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |
|                                  |                   |                                                |

