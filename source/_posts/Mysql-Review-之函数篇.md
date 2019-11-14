---
title: Mysql Review 之函数篇
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Mysql基础
tags:
  - Mysql
  - Mysql基础
  - 大熊BIGBEAR
translate_title: mysql-review-function
date: 2019-11-12 22:24:39
---


#### 数据处理函数
>Mqsql 和其他大多数计算机语言一样,支持利用函数处理数据,大多数SQL都支持以下类型的函数.
* 处理文本串的文本函数

* 数值数据上进行算术操作的数值函数
* 处理日期和时间值并从这些值中提取特定成分的日期和时间函数
* 返回DBMS正使用的特殊信息的系统函数

__1. 文本处理函数__
常用的文本处理函数一览 

| 函数 | 说明 | 示例 |
| ------ | ------ | ------ |
| Left() or Right() | 返回字符串左 or 右边的字符 |   |
| Length() | 返回字符串的长度 |  |
| Locate() | 找出字符串的一个子串 |  |
| Lower() or Upper() | 将字符串转换为小 or 大写 |  |
| LTrim() or RTrim() | 去掉字符串左 or 右边的空格 |  |
| Substring() | 截取字符串的子字符串 |  |

__2. 日期处理函数__
常用的日期和时间处理函数    

| 函数 | 说明 | 示例 |
| ------ | ------ | ------ |
| AddDate() or AddTime() | 增加一个日期 or 时间 |  |
| CurDate() or CurTime() | 返回当前日期 or 时间 |  |
| Date() | 返回日期部分 |  |
| DateDiff() | 计算两个日期之差 |  |
| Date_Format() | 返回一个格式化的日期或时间串 |  |
| DayOfWeek() | 返回一个日期,对应的是星期几 |  |

__3. 数值处理函数__
常用的数值处理函数

| 函数 | 说明 | 示例 |
| ------ | ------ | ------|
| ABS() | 返回绝对值 |  |
| Mod() | 返回除操作的余数 |  |
| Rand() | 返回一个随机数 |  |
| Sqrt() | 返回一个数的平方根 |  |