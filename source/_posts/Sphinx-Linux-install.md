---
title: Sphinx Linux 下安装过程
categories:
  - - 大熊BIGBEAR
    - Mysql
    - Sphinx
  - - 大熊BIGBEAR
    - PHP
  - - 大熊BIGBEAR
    - Linux
tags:
  - PHP
  - Mysql
  - Sphinx
  - Linux
  - Sphinxapi
  - 大熊BIGBEAR
translate_title: sphinx-linux-installation-process
date: 2019-11-12 23:17:58
---
<meta name="referrer" content="no-referrer" />

&emsp;&emsp;之前记录过一篇[Sphinx在Windows上的安装步骤](https://www.jianshu.com/p/1be12635ccbb),这篇当然就是Linux系统的安装步骤啦

<!-- more -->


__1. 下载sphinx包__
下载的是当前最新版3.1.1,我将压缩包存在了`/usr/local/src/`目录下
`wget -q http://sphinxsearch.com/files/sphinx-3.1.1-612d99f-linux-amd64.tar.gz ` 下载时间可能比较长需要耐心等待一会

![压缩包名称](https://upload-images.jianshu.io/upload_images/14618365-404c0f04014aa56d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


__2. 解压__
`tar zxf sphinx-3.1.1-612d99f-linux-amd64.tar.gz `解压
`mv sphinx-3.1.1 sphinx` 重命名一下,方便操作吧
`cd sphinx` 进入看下目录列表,其中var是需要后面创建的,请继续往下看
![解压后的目录](https://upload-images.jianshu.io/upload_images/14618365-33d59a1897fe1350.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__3. 编辑配置文件__
配置文件在`etc`目录下,有两个配置文件示例,其中`sphinx.conf`是完整版的配置项,并且有英文注释,有兴趣的可以了解一下,我们这里使用下面的`sphinx.conf.dist`简洁版
复制一份`sphinx.conf.dist`到`bin`目录下,重命名为`sphinx.conf`
`vi sphinx.conf` 打开编辑配置文件,这里我直接贴出我自己的配置内容,可以正常跑起来的,关于其他配置项可以看我的另外一篇[Sphinx 配置文件sphinx.conf配置项全解析](https://www.jianshu.com/p/be12aa194f15)
```

//数据源配置,也就是数据来源
source item
{
        type                    = mysql
        sql_host                = HOST //数据库
        sql_user                = USER //用户名
        sql_pass                = PASS //数据库密码
        sql_db                  = poster 数据库名称
        sql_port                = 3306  # optional, default is 3306
        sql_query_pre           = SET NAMES utf8 
        sql_query               = \
                SELECT id,name,UNIX_TIMESTAMP(ctime) \
                FROM item
        sql_attr_timestamp  = ctime
}

//索引配置
index item
{
        source                  = item //数据源
        path                    = /usr/local/src/sphinx/var/data 索引存放目录
        min_word_len            = 1 
        ngram_len               = 1
        ngram_chars             = U+3000..U+2FA1F
}


indexer
{
        mem_limit               = 128M
}

//搜索服务
searchd
{
        listen                  = 9312
        listen                  = 9306:mysql41
        log                     = /usr/local/src/sphinx/var/log/searchd.log
        query_log               = /usr/local/src/sphinx/var/log/query.log
        read_timeout            = 5
        max_children            = 30
        pid_file                = /usr/local/src/sphinx/var/log/searchd.pid
        seamless_rotate         = 1
        preopen_indexes         = 1
        unlink_old              = 1
        binlog_path             = /usr/local/src/sphinx/var/data
}

```
配置文件中需要配置几个目录.分别是__log__,__query_log__,__pid_file__,__binlog_path__,我们回到sphinx下,新建一个目录var,然后进入再新建一个data目录和一个log目录
`mkdir var`
`cd var`
`mkdir data`
`mkdir log`
如图:
![var目录](https://upload-images.jianshu.io/upload_images/14618365-252597381f2da18b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/14618365-87dec842ab8e3fc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 建立索引
首先进入到`bin`目录下,执行下面的指令
`./indexer -c sphinx.conf item` __item__为刚才在配置文件中建立的索引名称

>这里有个大坑,耽误了好久时间,我用的是ubuntu的系统,当我在执行`./indexer -c sphinx.conf item`时,出现了一个错误:___sql_connect: failed to load libmysqlclient (or libmariadb)___,加载libmysqlclient失败,然后我在/usr/lib/x86_64-linux-gnu/目录下发现了我的libmysqlclient,并不是没有安装的
![libmysqlclient](https://upload-images.jianshu.io/upload_images/14618365-05c1f3a9443d1e3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后我通过google试着找出问题的答案,然而只有两个相关结果,解决方式是建立软链接,并且将libmysqlclient所在的目录加入到环境变量,我照做了之后发现问题并没有顺利解决,最后,我试着下载所有的依赖
`apt-get install libmysqlclient18 libmysqlclient-dev libmysqlcppconn7 libmysqlcppconn-dev `
等待完成,然后忽略error,执行
`apt-get update`
等待结束后,再执行建立索引命令,解决了这个问题

__5. 开启搜索服务__
`./searchd`
