---
title: sphinx配置多索引
date: 2020-01-08 17:50:52
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Sphinx
tags:
  - PHP
  - Mysql
  - Sphinx
  - 大熊BIGBEAR
---

<meta name="referrer" content="no-referrer" />


__之前有写过sphinx的简单使用, 初次接触sphinx的同学可以先了解下, windows和linux都有__
* [Linux下安装sphinx及配置](http://blog.pulsating.cn/2019/11/12/Sphinx-Linux-install/)
* [Windows下安装sphinx及配置](http://pulsating.cn/2019/11/12/install-Sphinx-in-Windows-and-use-Sphinxapi-in-php/)
本文记录以下如何在sphinx的配置文件`sphinx.conf`中, 配置多个索引, 并启动服务
<!-- more -->

## 修改配置文件

#### 原配置文件
找到sphinx.conf配置文件, 假设当前已经有一个索引已经配置ok,  那么你的文件打开后应该类似这样的,其中数据源为article_source,对应文章表, 索引名称为article,对应article_source

```
source article_source
{
        type                    = mysql
        sql_host                = SQLHOST
        sql_user                = username
        sql_pass                = password
        sql_db                  = DBNAME
        sql_port                = 3306  # optional, default is 3306
        sql_query_pre           = SET NAMES utf8
        sql_query               = \
        SELECT id, title, content \
                FROM tablename
}

index article
{
        source                  = article_source # 对应上面source的名称
        path                    = /sphinx/article_data
        min_word_len            = 1
        ngram_len               = 1
        ngram_chars             = U+3000..U+2FA1F
}


indexer
{
        mem_limit               = 512M
}

searchd
{
        listen                  = 9312
        listen                  = 9306:mysql41
        log                     = /sphinx/searchd.log
        query_log               = /sphinx/query.log
        read_timeout            = 5
        max_children            = 30
        pid_file                = /sphinx/searchd.pid
        seamless_rotate         = 1
        preopen_indexes         = 1
        unlink_old              = 1
        binlog_path             = /sphinx/data
}


```

现在我们有一个新表, 假设为音乐表, 我们需要根据歌曲名称, 和歌词来匹配对应的歌曲, 供用户去搜索, 也需要使用sphinx, 那么我们应该如何做呢?

#### 构建新数据源
打开sphinx.conf文件,  在article_source数据源的下方, 声明下新的音乐数据源
```
source music_source
{
        type                    = mysql
        sql_host                = SQLHOST
        sql_user                = username
        sql_pass                = password
        sql_db                  = DBNAME
        sql_port                = 3306  # optional, default is 3306
        sql_query_pre           = SET NAMES utf8
        sql_query               = \
        SELECT id, name, lyric \
                FROM music
}
```

#### 配置index
在index article下方, 构建音乐数据源的索引
```
index music
{
        source                  = music_source # 对应上面source的名称
        path                    = /sphinx/music_data
        min_word_len            = 1
        ngram_len               = 1
        ngram_chars             = U+3000..U+2FA1F
}
```
__这里要注意一下, 两个索引的path不能在同一个目录下, 也有可能是我操作有误, 放在同一目录下面一直报错__

至此, 配置文件就配置好了

## 重新构建索引
#### 停止searchd服务
构建索引时需要停掉运行中的searchd服务, 不然会失败, 原索引如果正在使用中, 请谨慎停止, 或者研究一下`--rotate` 参数

__`--rotate` 用于轮换索引，在不停止服务的时候（searchd运行时）增加索引；searchd运行时不加会报错。__

__停止searchd命令:__
```
searchd -c source.conf --stop
```


#### 重建索引
__命令:__

```
indexer -c sphinx.conf -all
```

#### 启动searchd服务
```
searchd
```