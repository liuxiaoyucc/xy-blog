---
title: Mysql max_allowed_packet 设置
date: 2019-12-26 22:05:48
categories:
  - - 大熊BIGBEAR
    - Mysql
tags:
  - Mysql
  - 大熊BIGBEAR
---

> __max_allowed_packet__ 限制着你的mysql serve接收的数据包大小, 如果insert或者update时数据过大,超出max_allowed_packet的限制,mysql会抛出错误导致操作失败,那么我们来看下如何查看和设置max_allowed_packet
<!-- more -->
## 查看
```
mysql> show global variables like 'max_allowed_packet';
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| max_allowed_packet | 16777216 |
+--------------------+----------+
1 row in set, 1 warning (0.00 sec)
```
可以看到当前max_allowed_packet设置为16M (1024 * 1024 * 16)

## 修改

* 通过mysql命令修改(临时,重启mysql会重置为原始大小)
```
mysql> set global max_allowed_packet=32*1024*1024;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    6
Current database: mine

Query OK, 0 rows affected (0.01 sec)

mysql> show global variables like 'max_allowed_packet';
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| max_allowed_packet | 33554432 |
+--------------------+----------+
1 row in set, 1 warning (0.00 sec)

```
可以看到现在的value值已经变为33554432,也就是32M

* 修改mysql配置文件(不会失效,直到再次修改配置文件)
	1. windows下,找到mysql安装目录下__my.ini__文件
	2. 找到这一行__max_allowed_packet=16M__将16M修改为你想要的大小,这里改为8M,保存
	3. 重启mysql服务,查看当前max_allowed_packet大小,发现已经成功修改为8M
```
mysql> show global variables like 'max_allowed_packet';
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    2
Current database: mine

+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 8388608 |
+--------------------+---------+
1 row in set, 1 warning (0.01 sec)
```