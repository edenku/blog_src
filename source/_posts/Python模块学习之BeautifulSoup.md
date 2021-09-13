---
title: Python模块学习之BeautifulSoup使用
categories: Python
date: 2021-09-07
tags: Python
---

> BeautifulSoup简称Bs4，是爬虫非常常用的三方库。

[中文文档](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id7)

### 安装BS4

```shell
## pip 安装方式
$ pip install beautifulsoup4
$ pip install lxml
$ pip install html5lib
## easy_install
$ easy_install beautifulsoup4
$ easy_install lxml
$ easy_install html5lib
```

lxml 与html5lib都是第三方的HTML解析器。

### 快速开始

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>
<p class="story">
 	Once upon a time there were three little sisters; and their names were
 <a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
 <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
 <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
 and they lived at the bottom of a well.
</p>
<p class="story">...</p>
"""
```

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
```

简单使用

```powershell
>>> soup.title
<title>The Dormouse's story</title>
>>> soup.title.name
'title'
>>> soup.title.get_text()
"The Dormouse's story"
>>> soup.title.parent
<head><title>The Dormouse's story</title></head>
>>> soup.title.parent.name
'head'
```

找到所有a标签的链接

```powershell
>>> soup.a
<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
>>> m = [ o['href'] for o in soup.find_all('a') ]
>>> m
['http://example.com/elsie', 'http://example.com/lacie', 'http://example.com/tillie']
>>> n = [ o.get('href') for o in soup.find_all('a') ]
>>> n
['http://example.com/elsie', 'http://example.com/lacie', 'http://example.com/tillie']
```

### CSS选择器

```powershell
>>> soup.select("body p a")   # 逐级查找
>>> soup.select("p > a")	  # 直接找到标签下的子标签 
[<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
>>> soup.select("#link1 ~ .sister")	#后续兄弟节点选择器
[<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
<a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]
>>> soup.select("#link1 + .sister") # 相邻兄弟节点选择器
[<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

## 理论知识

### 对象种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种:Tag` , `NavigableString` , `BeautifulSoup` , `Comment.

```powershell
>>> type(soup.a)  # Tag
<class 'bs4.element.Tag'>
>>> soup.a.name	  # Name 每个tag都有自己的名字,使用.name获取
'a'
>>> soup.a		 # Attributes 一个tag可能多个属性，诸如 class/href/id等。
<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
>>> soup.a['href']
'http://example.com/elsie'
>>> soup.a['class']
['sister']
				# Comment是一个特殊类型对象 像注释..
>>> markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
>>> soup = BeautifulSoup(markup)
>>> comment = soup.b.string
>>> type(comment)
# <class 'bs4.element.Comment'>
```





