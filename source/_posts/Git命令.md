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

### reset

#### gitlab回滚代码

gitlab主master分支默认受保护，不允许任何人进行代码回滚，所以我们回滚需要先删除受保护标签。

![image-20210629104157190](C:%5CUsers%5Cci22578%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210629104157190.png)

project-setting 去设置

![image-20210629104419193](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210629104419193.png)

点击不再保护按钮。

![image-20210629104530667](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210629104530667.png)

接下来，我们就可以回滚到我们指定的代码提交记录了。

```powershell
git reset --hard <commit-id>
git push -f   ## 强制推送
## 输入用户密码即可
```

操作完以后最好还是把这个保护再添加回来。

### tag

```
# 打标签
git tag -a V1.0 -m 'V1.0的描述'
# 删除标签
git tag -d V1.0
# 推送远程
git push origin V1.0
```

