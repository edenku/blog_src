---
title: Jenkins用户权限控制
date: 2021-10-29
categories: Jenkins
tags: Jenkins

---

> 根据各人员角色定位不同，为每个角色分配不同的数据权限，诸如：研发人员分配 dev的构建权限，测试人员分配 test 构建权限，运维 分配生产环境 构建权限。

### 插件安装

![image-20211029151049314](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151049314.png)

进入Jenkins的插件管理，找到`Role-based Authorization Strategy`插件。点击安装。

![image-20211029151139108](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151139108.png)

安装完成效果如下。

![image-20211029151245929](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151245929.png)

进入，全局安全配置，配置授权策略。(我第一次安装，貌似没安装上，看不到选项，重新安装一次就可以了)。

![image-20211029151651816](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151651816.png)

保存。就可以看到这了有一个Manage and Assign Roles了。

![image-20211029151822998](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151822998.png)

### 添加用户

点击上图的Manage Users添加几个用户。我这里建一个dev、一个test。

![image-20211029151931777](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029151931777.png)

### 权限配置

进入刚才的Manage and Assign Roles。我们先管理角色。

![image-20211029152007031](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029152007031.png)

刚进来是这个样子

![image-20211029152053333](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029152053333.png)

这里比较绕，我们先说明下。

+ Global roles：基础角色，具体啥样不太清楚。
+ Item roles：实际控制权限的角色。

![image-20211029152423390](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029152423390.png)

我们按百度的文档，先添加一个base-role，给全局read权限，再分别建立一个dev和test。

> 这里的Pattern是角色做过滤的，比如test.*,会把所有Job名称test开头的Job，分配给test角色。

然后回到刚才的Manage and Assign Roles。我们开始分配角色。

![image-20211029152605218](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029152605218.png)

给dev和test都授予base的全局角色，分别给与dev和test对应的item-role。记得保存。

>  注意：这里的用户是在下面文本框输入的。不是选择的。

接下来我创建4个测试Job，dev1/dev2/test1/test2

然后分别登陆dev和test 看看，是不是已经ok了？

![image-20211029153247587](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211029153247587.png)

> 如果，发现permission dennied字样，回头看看上面哪一步是否忘记保存了。