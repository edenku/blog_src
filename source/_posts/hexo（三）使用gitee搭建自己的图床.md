---
title: hexo（三）使用gitee搭建自己的图床
categories: 使用hexo搭建博客
date: 2021-05-11
tags: hexo
---

## gitee创建个人repo

使用gitee创建一个public仓库image_repo

## 下载picGo

  [官网](https://molunerfinn.com/PicGo/) 下载picGo，windows直接选择exe版本。

![image-20210511164334718](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511164334718.png)

下载完成，傻瓜安装完成即可。

<!-- more -->

### 创建私人令牌

头像 --> 设置 --> 私人令牌 --> 生成令牌

![image-20210511164833637](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511164833637.png)

输入密码，生成令牌，复制后面备用。

![image-20210511164930817](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511164930817.png)

如果没有复制下来，可以点击修改，重新生成一个。

### picGo配置

+ 安装gitee插件

  ![image-20210511165157933](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511165157933.png)

  安装后重新打开软件，图床设置，找到gitee菜单。

  ![image-20210511165510260](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511165510260.png)

  repo设置右上角圈出的部分。token使用我们上面生成的令牌。然后确定，并设置为默认图床。

+ 使用

  接下来就可以使用picGo上传图片了。

  ![image-20210511165940775](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511165940775.png)

  

  上传区上传图片，相册菜单负责管理图片。

### Typora配置picGo

> typoRa是一个极致简洁的markdown编辑器，即时渲染。

下载链接: https://www.typora.io/#windows

一路默认安装，打开软件后，文件 --> 偏好设置。

![image-20210511170603136](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511170603136.png)

根据图中配置后，可以验证图片上传。出现下图即为配置成功。

![image-20210511170650016](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210511170650016.png)



### 总结

到此一个免费的博客图床就配置好了。

