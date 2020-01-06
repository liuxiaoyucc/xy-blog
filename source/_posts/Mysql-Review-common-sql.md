---
title: Mysql Review 之常用sql语句
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Mysql基础
tags:
  - Mysql
  - Mysql基础
  - 大熊BIGBEAR
translate_title: mysql-review-common-sql-statement
date: 2019-11-12 22:13:03
---

### 一. 综合
1. 使用__.sql__脚本 ``` SOURCE + 脚本路径;```
2. 打开数据库 ``` USE database; ```
3. 显示所有数据库名称 ```SHOW databases; ``` 
4. 显示所有表名称 ```SHOW tables; ```
5. 显示表中的列及属性 ```SHOW column FROM table_name;```

### 二. 检索/查询语句
##### 普通查询
1. 检索某一列 ```SELECT column_name FROM table_name;```
2. 检索所有列,使用 * 通配符 ```SELECT * FROM table_name;```
##### DISTINCT
3. 检索结果去重 ``` SELECT DISTINCT column_name FROM TABLE_NAME```
##### LIMIT
4. 限制检索结果行数
* ``` SELECT column FROM table_name LIMIT 5 ## 结果不超过5行```
* ``` SELECT column FROM table_name 5,5 ## 返回从行5开始的5行```
* ``` SELECT column FROM table_name 5 OFFSET 4 ## limit m,n的替代语法,返回5行,从行4开始```  
##### ORDER BY
5. 检索结果排序
* ```SELECT * FROM table_name ORDER BY id DESC 降序/倒序```
* ```SELECT * FROM table_name ORDER BY id ASC ## 升序/正序``` 
6. 按多个列排序```SELECT * FROM table_name order by column1,column2;## 先按column1 排序,如果column1有2个或多个值相同,则再按照column2排序```

##### WHERE子句
7. ```SELECT * FROM table_name WHERE price = 2.5;## 检索出价格为2.5的产品```

8. WHERE 子句操作符

| 操作符 | 说明 |
| ------ | ------ |
| = | 等于 |
| <> | 不等于 |
| != | 不等于 |
| < | 小于 |
| <= | 小于等于 |
| > | 大于 |
| >= | 大于等于 |
| BETWEEN m AND n | 指定的两个值之间 |

9. 空值检查 ```WHERE column  IS NULL; ## 特殊的where 子句```

10. 逻辑操作符(operator)
* and  ``` WHERE age = 19 AND score > 80 ; ```
* or ```WHERE user_id = 1 or user_id = 2;```  
* in ```WHERE user_id in (1,2,3,4);```
* not 否定后跟条件的关键字

11.使用通配符进行过滤
通配符(wildcard) 用来匹配值的一部分的特殊字符.
搜索模式(search pattern) 由字面值,通配符,或两者组合构成的搜索条件.
* 百分号%通配符,可以在搜索模式的任意位置使用任意个通配符,
 ``` 
 SELECT * FROM table_name WHERE username LIKE 'rise%'; ##检索以rise开头的用户名 
 ```
* 下划线_通配符,用途和%一样,但是只能匹配单个字符而不是多个
``` 
SELECT * FROM table_name WHERE price LIKE '_000'; ##检索出价格为1000,2000等的商品 
```

不要过度使用通配符,并且尽量不要在搜索模式的开始出使用,搜索速度巨慢!

11. Mysql使用正则表达式
```
SELECT * FROM table_name WHERE prod_name REGEXP '正则表达式'; 
```

##### 计算字段
12. __Concat()__拼接两个列   

```
## 将两个列拼接起来,变成一个新字段vend-title,类似  '值name(location) '的形式,同时去掉两个字段右侧的空格.
## RTrim(),LTrim(),Trim()
## AS 用as赋予别名 (又叫导出列)
SELECT Concat(RTrim(vend_name) , ' (', RTrim(vend_country), ') ') AS vend_title 
FROM vendors
ORDER BY vend_name;
```

13. 执行算术计算
```
##通过单价和数量两个列运算出商品总价并形成一个新字段 expanded_price
SELECT prod_id,
       quantity,
       item_price,
       quantity * item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```
14. 测试计算

```
SELECT 3 * 2;
SELECT Trim('abc');
SELECT Now();
```