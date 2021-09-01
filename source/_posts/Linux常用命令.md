---
title: Linux常用命令
date: 2021-05-14
cateogries: 常用命令
---

###  防火墙

**查看防火墙状态**

systemctl status firewalld.service

firewall-cmd --state

**关闭防火墙**

systemctl stop firewalld.service

**开启防火墙**

systemctl start firewalld.service

**禁用防火墙**

systemctl disable firewalld.service

<!-- more -->

**设置开机自动启动**

systemctl enable firewalld.service

 **关闭开机启动**

systemctl disable firewalld.service

**查看防火墙所有开放的端口**

firewall-cmd --zone=public --list-ports

### 查看系统版本

cat /etc/redhat-release

### 常见插件安装

#### sz 和 rz安装

yum install lrzsz

### 删除乱码文件

```shell
ls -i   # 显示文件的节点号
find -inum 3031169 -delete
```

再看就已经成功删除了。

![image-20210530221326703](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210530221326703.png)

### 查看服务器占用空间

**df**

df可以查看根目录大小、使用比例及其挂入点。

![image-20210617094210820](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210617094210820.png)

**du**

du可以查看文件及文件夹的大小。但目录层级过多的时候，列表数目过多并不利于查看。所以我们可以指定目录深度 

```
du -h --max-depth=1 /usr/local
```



![image-20210617095002419](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210617095002419.png)

### 文件查找

```
find / -name http.conf  	# 在根目录下查找文件http.conf
find / -name 'name'		    # 在根目录查找文件名包含name的文件

```

