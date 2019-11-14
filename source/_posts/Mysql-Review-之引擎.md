---
title: Mysql Review 之引擎
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Mysql基础
tags:
  - Mysql
  - Mysql基础
  - 大熊BIGBEAR
translate_title: the-engine-of-mysql-review
date: 2019-11-12 22:31:31
---

**1. Mysql具有多个引擎,都隐藏在Mysql服务器内,全都可以执行CERATE TABLE 和SELECT等命令**
__2. 不同的引擎有不同的功能和灵活性,下面列出几个必须要知道的引擎:__
* __InnoDB__ 是一个可靠的事务处理引擎,不支持全文本搜索.__
* __MEMORY__ 功能上等于MyISAM,但由于数据存储在内存(不是磁盘),速度很快,特别适用于临时表.
* __MyISAM__ 性能极高,支持全文本搜索.但是支持事务处理.

__3. 引擎类型可以混用__

>注:外键不可以跨引擎,即使用一个引擎的表不能引用具有不用引擎的表的外键,这也是混用引擎的一大缺陷