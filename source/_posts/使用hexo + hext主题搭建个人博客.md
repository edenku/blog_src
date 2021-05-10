---
title: hexo（一）:使用hexo + hext主题搭建个人博客
categories: hexo
---

### 准备工作

#### 安装nodejs

  从[官网](https://nodejs.org/en/download/)下载最新的安装版本。Windows Installer (.msi) 64-bit
 
  这里也可以从[淘宝镜像](https://npm.taobao.org/)去下载。
 
  一路默认下一步安装。
  
#### 安装Hexo

 `npm install -g hexo`全局安装hexo。

 报错信息：

 Q：`npm WARN deprecated titlecase@1.1.2: no longer maintained`
 
 A：`npm -v`查看npm版本，`npm install -g npm `更新版本即可。
 
<!-- more -->

#### 准备github帐号

注册github帐号

新建一个repository，名称为 `yourname.github.io`，我的帐号名为`e***u`，故我的repository为`e***u.github.io`。

#### 安装git.
 
 从[官网](https://git-scm.com/download/win)选择64位下载。
 
 一路默认下一步安装。

### 创建本地博客 

#### 环境检测

```
node -v // 检测node是否正常安装
hexo -v // 检测hexo是否正常安装
npm -v  // 检测npm 是否正常安装
git --version //检测git版本
```

#### 初始化项目

   ```
   D:\Eden\gitee-blog> hexo init 2021-blog
   ```
   
   执行命令时，报”无法加载文件 C:\Users\ci22578\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。“。
   
   以管理员身份运行 powershell, 并执行 set-ExecutionPolicy RemoteSigned 命令，将计算机上的执行策略更改为 RemoteSigned，即可。
   
#### 启动
  
  ```
  D:\Eden\gitee-blog\2021-blog> hexo s [-p 端口号]  //启动，默认端口4000
  ```

#### 修改博客基本信息

	修改博客根目录下的_config.yml文件，后面称站点配置。

  ```
  # Site 博客主配置信息
	title: Hexo                             # 博客名称
	subtitle: '一条瞎扑腾的咸鱼'            # 博客子标题
	description: '一条瞎扑腾的咸鱼'         # 作者描述
	keywords:                               # 站点关键字，用于搜索优化
	author: Eden 懒散老马                   # 博主名
	language: zh-CN                         # 站点语言
	timezone: 'Asia/Shanghai'               # 时区	 
  ```

### 配置next主题
  
#### 下载next主题。
  
  git clone https://github.com/theme-next/hexo-theme-next themes/next
  
  访问不了也可以在gitee上搜索别人同步过来的。
  
  如果在博客根目录按上面命令下载，则直接跳过。如果是在其他目录下载，则将下载文件夹更名为next，拖放到博客目录中themes目录下。
  
#### 应用next主题
  
	修改站点配置（2021-blog\_config.yml)。
	
  ```
  # Extensions 扩展信息配置
	theme: next	                            # 主题设置为我们的next主题，默认是landscape
  ```
  
### next扩展配置。


#### 主题样式配置


	修改主题目录下的_config.yml配置
  
  ```
  # ---------------------------------------------------------------
  # Menu Settings 菜单设置
  # ---------------------------------------------------------------
  menu:
  home: / || fa fa-home
  #about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
  
  # ---------------------------------------------------------------
  # Scheme Settings 样式设置
  # --------------------------------------------------------------- 
  # Schemes
  #scheme: Muse
  #scheme: Mist
  scheme: Pisces 
  #scheme: Gemini

  # Dark Mode
  darkmode: false
  
  ```
  
#### 隐藏底部next / 强力驱动字样。
  
  themes\next\layout\_partials目录中找到footer.njk（不同版本的文件扩展名可能不同）
  
  ```
	用 <!-- --!> 包住下面这段代码 或 直接删除
	<!--
	{%- if theme.footer.powered %}
	  <div class="powered-by">
		{%- set next_site = 'https://theme-next.js.org' if theme.scheme === 'Gemini' else 'https://theme-next.js.org/' + theme.scheme | lower + '/' %}
		{{- __('footer.powered', next_url('https://hexo.io', 'Hexo', {class: 'theme-link'}) + ' & ' + next_url(next_site, 'NexT.' + theme.scheme, {class: 'theme-link'})) }}
	  </div>
	{%- endif %}
	--!>
  ```
  
#### Local Search 本地搜索
  
  安装插件 hexo-generator-searchdb
  
  ```
   npm install hexo-generator-searchdb --save
  ```
  
  修改根目录_config.yml，添加下面配置到文件末尾。
  
  ```
  search:
    path: search.xml
    field: post
    format: html
    limit: 10000
  ```
  
  修改主题配置_config.yml,启用本地搜索
  
  ```
  # Local Search
  # Dependencies: https://github.com/next-theme/hexo-generator-searchdb
  local_search:
    enable: true
  ```
  
#### 设置网站图标
  
  去easyicon找个图标
  
  ```
  favicon:
    small: /images/favicon_48px.ico
    medium: /images/favicon_48px.ico
    apple_touch_icon: /images/apple-touch-icon-next.png
    safari_pinned_tab: /images/logo.svg
  ```
  
#### 增加文章字数统计和阅读时长
  
  Dependencies: https://github.com/next-theme/hexo-word-counter 目前设置没有生效，后续跟踪查看。
  
#### 博客加妹子
  
  npm install -save hexo-helper-live2d
  
  修改站点配置文件
  
  ```
  live2d:
    enable: true
    scriptFrom: local
    pluginRootPath: live2dw/
    pluginJsPath: lib/
    pluginModelPath: assets/
    tagMode: false
    log: false
    model:
			use: live2d-widget-model-shizuku
    display:
      position: right
      width: 150
      height: 300
    mobile:
      show: true
  ```
  
  安装模型，在命令行运行命令：npm install --save live2d-widget-model-shizuku
	
	可选模型：
	
	live2d-widget-model-chitose

	live2d-widget-model-epsilon2_1

	live2d-widget-model-gf

	live2d-widget-model-haru/01 (use npm install --save live2d-widget-model-haru)

	live2d-widget-model-haru/02 (use npm install --save live2d-widget-model-haru)

	live2d-widget-model-haruto

	live2d-widget-model-hibiki

	live2d-widget-model-hijiki

	live2d-widget-model-izumi

	live2d-widget-model-koharu

	live2d-widget-model-miku

	live2d-widget-model-ni-j

	live2d-widget-model-nico

	live2d-widget-model-nietzsche

	live2d-widget-model-nipsilon

	live2d-widget-model-nito

	live2d-widget-model-shizuku

	live2d-widget-model-tororo

	live2d-widget-model-tsumiki

	live2d-widget-model-unitychan

	live2d-widget-model-wanko

	live2d-widget-model-z16

#### 代码复制
	
	```
	codeblock:
		border_radius: 8   # 按钮圆滑度
		copy_button:  # 设置是否开启代码块复制按钮
			enable: true
			show_result: true  # 是否显示复制成功信息
	```

### 总结

  本文我们从零开始搭建个人本地hexo博客，并使用next主题。后续我们尝试将hexo博客推送至github、gitee等站点。生成我们的博客站。
	
### 参考链接

  + [Hexo-Next 主题博客个性化配置超详细，超全面](https://blog.csdn.net/as480133937/article/details/100138838/)