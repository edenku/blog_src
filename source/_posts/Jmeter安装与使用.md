---
title: Jmeter安装与使用
date: 2021-05-28
categories: 软件安装
tags: Jmeter
---

### Jmeter的安装

#### 下载

[[官网](http://jmeter.apache.org/download_jmeter.cgi)]

![image-20210528163050542](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528163050542.png)

下载后直接解压即可。

<!-- more -->

其他博客讲要配置Jmeter的环境变量，我这里没有配置，直接运行jmeter.bat启动成功了。

![image-20210528163522940](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528163522940.png)

#### 设置中文

Options -> Choose Language -> Chinese(Simplified)

![image-20210528163625845](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528163625845.png)

### 简单示例

右键Test Plan创建一个线程组

![image-20210528164520982](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528164520982.png)

线程组包含线程数、Ream-Up时间、循环次数等设置。暂时我们使用默认值。

![image-20210528164759882](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528164759882.png)

右键线程组，添加取样器，选中HTTP请求

![image-20210528164909173](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528164909173.png)

我们配置了一个简单的HTTP请求：

![image-20210528164947516](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528164947516.png)

然后右键线程组，添加监听器选择一个 查看结果树

![image-20210528165023537](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528165023537.png)

选中我们的线程组，点击上方绿色执行按钮，执行。

![image-20210528165133177](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528165133177.png)

然后选择查看结果树，就可以看到执行结果了

![image-20210528165204344](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528165204344.png)

是不是很方便。

**修改编码**

仔细看会发现我们的响应中有乱码消息，这个是Jmeter默认ISO-9959-1编码造成的。关闭软件，找到bin目录下的jmeter.properties，找到sampleresult.default.encoding这个，放开注释，将ISO-9959-1替换为UTF-8,如图：

![image-20210528165440684](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528165440684.png)

重启Jmeter再执行乱码问题已经处理掉了。

![image-20210528165611784](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210528165611784.png)