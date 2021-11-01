---
title: MySQL常用命令
date: 2021-09-07
categories: 常用命令
tags: MySQL

---

1. 统计近7天的某数据，无数据时补0。

   ```sql
   SELECT
   	@cdate := date_add( @cdate, INTERVAL - 1 DAY ) date 
   FROM
   	( SELECT @cdate := date_add( CURDATE( ), INTERVAL 1 DAY ) FROM rp_notice LIMIT 7 ) a
   ```

2. MySQL 根据JSON字段内容进行判断

   ```sql
   select *
   from tb_review a 
   where json_unquote(json_extract(a.json_content,'$."ext:base:isA"')) = '否'
   -- JSON字段要用双引号 引起来，且用json_unquote去除引号
   ```

   