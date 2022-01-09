---
title: mysql 从树形结构表中获取叶子结点数据
date: 2020-06-20 09:59:36
categories:
  - - 大熊BIGBEAR
    - Mysql
tags:
  - Mysql
  - 大熊BIGBEAR
---

<meta name="referrer" content="no-referrer" />
> 叶子结点 就是出度为0的结点 就是没有子结点的结点

<!-- more -->
先记录下sql, 后面详细说明

catalogue是一张存储着书籍目录的表

主要字段包括

* id 主键, 唯一 

* parent_id 父目录id

* book_id 对应的书籍id

```mysql
select b.* 
from catalogue a 
right join catalogue b 
on a.parent_id = b.id
where b.book_id = 555
group by b.id 
having count(a.id) = 0;
```

