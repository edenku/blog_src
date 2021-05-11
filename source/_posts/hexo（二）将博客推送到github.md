---
title: hexo（二）:将博客推送至github
categories: hexo
---


### 操作步骤。

 登录github，并创建public仓库。
 
 仓库名命名格式：如果帐号名为`jack`，则repository为`jack.github.io`
 
### 安装deploy扩展

在跟目录(2021-blog)内，执行`npm install hexo-deployer-git --save` 安装部署的扩展。

### 修改站点配置

```xml
deploy:
  type: git
  repo: git@github.com:jack/jack.github.io.git
  branch: master
```

<!-- more -->

### 推送

 根目录下使用 hexo d -g 即可将项目推送至github远程仓库。
	
 快使用jack.github.io看看能不能访问了？

### github添加免密

 本地生成ssh秘钥文件

 ```
	// 检查是否存在
	cd ~/.ssh
	ls
	// 生成密钥，一路回车即可
	ssh-keygen -t rsa -C “m***n_11@126.com” 
 ```
 
 github网站，头像-> settings-> SSH and GPG keys -> New SSh Key 
 
 将电脑上C:\Users\.ssh 中的id_rsa.pub中的内容 复制到key中，title自定义即可。
 
 添加完毕后，本地命令行 输入 ssh -T git@github.com 测试下链接，当看到
 
 `Hi jack! You've successfully authenticated, but GitHub does not provide shell access.`
 
 说明配置成功，再推送就可以免密推送了。
 
 >> 我这里后来推送，免密没有成功。源于我自己github多个账号的原因，所以这里需要重新调整下

### github多账号配置

 