---
title: 使用命令行执行CI控制器中的函数
date: 2020-01-07 23:29:56
categories:
  - - 大熊BIGBEAR
    - PHP
    - CI
tags:
  - PHP
  - CI
  - 大熊BIGBEAR
---

<meta name="referrer" content="no-referrer" />

> 有些时候我们在执行一些长时间的任务的时候, 使用浏览器运行是很不友好的, 看不到实时输出, 时间过长浏览器还会超时, 这时候就需要在命令行中来执行

<!-- more -->

## 调用方式

```
php index.php controller method param
```

## tips: 

* 需要将php路径设置到环境变量中, 或者指定php路径
