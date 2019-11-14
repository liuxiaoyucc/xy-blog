---
title: Mysql Review 之表联结入门篇
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Mysql基础
tags:
  - Mysql
  - Mysql基础
  - 大熊BIGBEAR
translate_title: mysql-review-table-connection-entry
date: 2019-11-12 22:30:06
---
<meta name="referrer" content="no-referrer" />

#### 1. 为什么要使用联结?

    联结可以利用一条SELECT语句检索出存储在多个关系表上的数据,也就是联结多个表返回一组数据.

#### 2. 一个创建联结的简单例子
```
SELECT vend_name,prod_name,prod_price //检索出这三列数据
FROM vendors,products //从这两个表中
WHERE vendors.vend_id = product.vend_id //条件是两个表中的vend_id相等
ORDER BY vend_name,prod_name; //通过vend_name,prod_name排序
```
创建联结语句时,一定要注意的WHERE子句,如果没有WHERE子句,第一个表中的每一行都i将于第二个表中的行配对,从而产生大量不希望检索出的结果,也就是__笛卡儿积__(没有联结条件的表关系返回的结果)

___应该保证所有联结都有WHERE子句,并且保证WHERE子句的正确性___

