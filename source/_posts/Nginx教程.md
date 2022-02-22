---
title: Nginx教程
categories: Nginx
tags: Nginx
date: 2022-01-07
---

## Nginx教程

> Nginx是俄罗斯人Igor Sysoev（伊戈尔.塞索耶夫)编写的一款高性能HTTP和反向代理服务器。

### 安装与配置文件

#### Windows安装

官网[http://nginx.org/](http://nginx.org/)地址。将zip包安装至没有空格的路径，解压后，cmd中执行`start nginx`即可。

启动后访问 localhost/ 可以看到nginx默认的http页面。

![image-20220107175454713](https://gitee.com/ruocy/image_repo/raw/master/images/image-20220107175454713.png)

#### Linux安装

//TODO...

### 配置

nginx的配置文件默认在安装目录的conf目录下，主配置文件为nginx.conf.

#### 全局配置

```nginx
# 使用的用户和组
#user  nobody;
# 指定工作进程数（一般为CPU核数或核数*2）
worker_processes  1;
# 错误日志存放路径，错误级别[debug| info | notice| warn| error| crit]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
# pid存放路径
#pid        logs/nginx.pid;

# 设定Nginx工作模式及连接数上限
events {
    # use epoll;  指定工作模式，可选[select | poll| kqueue| epoll 等..] 首选epoll
    worker_connections  1024; 最大连接数设置，默认1024
}
```

#### HTTP服务器配置

```nginx
http {
	# include属主模块指令，实现对配置文件所包含文件的设定，可减少主配置文件的复杂度。
		# mime.types 是nginx.conf同级文件，web服务器收到静态资源文件，根据文件后缀名在mime.types中找到对应的mime type,设置content-type，浏览器根据content-type处理文件。
    include       mime.types; 
    # 默认的类型
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
	# 开启高效文件传输模式 普通应用设为on，磁盘IO重负载应用，可设置为off以平衡磁盘和网络IO处理速度，降低负载
    sendfile        on;
    #tcp_nopush     on;
	# 长连接超时时间，单位是秒，这个参数很敏感
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	# server虚拟主机
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

+ keepalive_timeout 长连接超时时间，单位秒。

### 配置梳理

