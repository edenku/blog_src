---
title: Linux编写Shell脚本
date: 2021-11-01
cateogries: Linux
---

### 入门实例

```shell
#!/bin/bash
echo "Hello World !"
```

### 脚本描述

#### 变量定义

```shell
#!/bin/bash
date=`date +%Y%m%d`
echo $date   #$加变量名 就可以使用
echo ${date} #也可以加上花括号标识变量的边界
readonly date # 标识变量为只读变量
```

> 1. 定义变量不加$符号。
> 2. 变量名与等号之间不能有空格。
> 3. 变量名命名使用英文、数字下划线，数字不能开头。
> 4. 不能使用关键字命名。

#### Shell字符串

```shell
str='this is a str'			# 可以使用单引号，单引号变量无效
str2="this is a str too"	# 也可使用双引号，双引号可以使用变量

str3="abcd"
echo ${#str3}               # 输出4，字符串的长度
echo ${str3:1:2}			# 输出bc，从idx为1开始截取2个字符
```

#### 流程控制

```
if condition
then
  command1
elif condition
then
  command2
else
  command
fi
```

### 简单的一个启停脚本

```shell
#!/bin/bash
# 备份
date=`date +%Y%m%d`
if [ ! -d "/data/bak/${date}" ]
then
mkdir /data/bak/${date}
fi
cp -rf /home/tomcat/webapps/java-pro.war "/data/bak/${date}"
echo $date
# 杀掉原始进程
pid=$(ps -ef|grep java-pro|grep -v grep|awk '{print $2}')
kill -9 $pid
echo $pid
# 程序更新
rm -rf /home/tomcat/webapps/java-pro.war
rm -rf /home/tomcat/webapps/java-pro
mv /home/tmp/java-pro.war /home/tomcat/webapps/
# 重启
/home/tomcat/bin/startup.sh
```

