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

2. todo 