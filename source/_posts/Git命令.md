---
title: Git命令整理
tags: git
date: 2021-05-24
categories: 常用命令
---

### clone

#### 下载指定tag项目

 ```
 # git clone --branch [tags标签] [git地址]
 git clone --branch dubbo-2.5.4 https://gitee.com/apache/dubbo.git
 ```

### push

#### 本地项目推送远程

```shell
1. git init 初始化本地项目
PS D:\code\demo> git init
PS D:\code\demo> git add .
PS D:\code\demo> git commit -m 'init'
2. 远程gitee/github创建空分支
3. 本地关联远程分支
PS D:\code\demo> git remote add origin http://xx.xx.xx.xx:port/demo/demo.git
4. 推送远程
PS D:\code\demo> git push -u origin master
```

