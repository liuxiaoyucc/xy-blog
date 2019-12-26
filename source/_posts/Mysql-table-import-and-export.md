---
title: Mysql table import and export 快速导出导入
date: 2019-12-20 21:44:13
categories:
  - - 大熊BIGBEAR
    - Mysql
tags:
  - Mysql
  - 大熊BIGBEAR
---

<!-- <meta name="referrer" content="no-referrer" /> -->

## 导出

{% codeblock %}
mysqldump -u root -p -q -e -t  db_name table_name > table_name.sql
{% endcodeblock %}


* –quick，-q
该选项在导出大表时很有用，它强制 mysqldump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。

* --extended-insert, -e
使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项。

* -t 
仅导出表数据，不导出表结构

## 导入

{% codeblock %}
use db_name;
source table_name.sql;
{% endcodeblock %}