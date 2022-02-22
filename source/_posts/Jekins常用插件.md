---
title: Jenkins常用插件
date: 2021-10-29
categories: Jenkins
tags: Jenkins

---

### Git Parameter

> 可以指定分支或标签，构建。

**安装插件**

![image-20211029154104897](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029154104897.png)

构建项目时，添加参数，选择Git参数

![image-20211029154158614](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029154158614.png)

设置一个参数变量，branch。

![image-20211029154349569](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029154349569.png)

这里分支我过滤了一下，`origin.*/(RELEASE.*)` 过滤出RELEASE开头的分支选项。

然后源码管理处，指定上面设置的变量branch。

![image-20211029154643491](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029154643491.png)

配置完的效果如下：

![image-20211029154752941](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029154752941.png)

> 我配置完后，第一次构建失败了。最初我配置的变量名为choose-branch,没识别出来，修改后ok了。

```
14:45:07  > git rev-parse origin/${choose-branch}^{commit} # timeout=10
14:45:07  > git rev-parse ${choose-branch}^{commit} # timeout=10
14:45:07 ERROR: Couldn't find any revision to build. Verify the repository and branch configuration for this job.
14:45:07 Finished: FAILURE
```

### Pubilsh Over SSH

安装插件

![image-20211029164646857](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029164646857.png)

系统配置

![image-20211029164907491](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029164907491.png)

配置SSH server

![image-20211029165856382](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029165856382.png)

我们使用username/password方式，点击高级，输入密码。然后Test Configuration测试能否连上远程服务。

![image-20211029170005698](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029170005698.png)

配置成功的话，可以看到Success

![image-20211029170029596](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029170029596.png)

**配置远程部署**

在Job的配置中，配置构建后操作。选择 Send Build artifacts over SSH。

> Maven项目有，其他项目不一定有

![image-20211029170235417](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029170235417.png)

