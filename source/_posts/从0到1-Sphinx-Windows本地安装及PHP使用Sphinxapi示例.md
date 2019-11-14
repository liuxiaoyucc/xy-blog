---
title: '从0到1,Sphinx Windows本地安装及PHP使用Sphinxapi示例'
date: 2019-11-12 23:01:44
categories: 
    - [大熊BIGBEAR, Mysql, Sphinx]
    - [大熊BIGBEAR, PHP]
tags: 
    - PHP
    - Mysql
    - Sphinx
    - Sphinxapi
    - 大熊BIGBEAR
toc: true
---
<meta name="referrer" content="no-referrer" />

&emsp;&emsp;最近一个项目需要实现这样一个需求:mysql数据库一张表中存了百万张菜品图片,需要根据菜品名称或描述,模糊匹配出符合条件的菜品图片,并展示出来

```

select * from table_name where column like '%鱼香肉丝%';

```

&emsp;&emsp;如果像上面那样,直接使用mysql like查询的话,会进行全表扫描,不走索引,大大的影响查询效率,所以开始学习使用__Sphinx全文搜索引擎__,下面记录下第一次配置使用的过程,以及过程中遇到的问题和疑问,望指正.

<!-- more -->


###### 1. 下载sphinx源码压缩包[点击下载](http://sphinxsearch.com/downloads/current/)
按照自己的需求,下载对应的版本,我这里下载的是Windows x64 binaries 3.1.1 版本

###### 2. 下载后的sphinx源码目录
![sphinx源码目录结构](https://upload-images.jianshu.io/upload_images/14618365-57bc4bf34fe6ce13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
###### 3.配置文件
打开etc目录,里面有三个文件,example,sql是创建测试表的脚本,我们稍后会用到,sphinx.conf.dist是完整版默认配置,这里我选择了sphinx-min.conf.dist简化版,暂时可以满足需求,复制一份到bin目录下,并且重命名为sphinx.conf.
![etc目录](https://upload-images.jianshu.io/upload_images/14618365-eeada956c4180e1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
###### 4. sphinx.config文件配置
配置文件的几个组成部分:
* source 数据源,这里配置你要进行全文检索的数据的来源
* index 索引,从上面的数据源处构建索引,相当于字典检索,要有整本字典内容后才会有索引
* indexer 构建索引,需要重新构建索引时,其实就是调用indexer这个命令
* searchd 提供搜索查询的服务,后台运行
```
source src1 ## src1为数据源命名的名称,可以根据数据源的属性更改
{
    type            = mysql ## 数据源,本项目使用mysql
    sql_host        = localhost ## 数据库服务器,这里因为是测试,使用的是本地数据库
    sql_user        = root # 数据库用户名
    sql_pass        = root # 数据库密码
    sql_db          = hongbao ## 数据库名称,替换成你自己的数据库名称
    sql_port        = 3306 ## 数据库端口,默认3306
    sql_query_pre   = SET NAMES utf8 ## 如果你的数据库不是uft8编码的,注释掉本行
    sql_query       = \ ## 主查询,查询出所有在检索范围的数据
        SELECT id, group_id, UNIX_TIMESTAMP(date_added) AS date_added, title, content \
        FROM documents
    sql_attr_uint       = group_id ## 属性
    sql_attr_timestamp  = date_added ## 属性,可用来排序
}

index test1 ## 索引名称,自行命名
{
    source          = src1 ## 基于这个数据源构建索引
    path            = D:/sphinx/data/ ## 存放索引的目录,自己创建
    charset_table = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F  ##  指定utf-8的编码表
    mlock           = 0
    morphology      = none ## 词形处理器,如果检索中文用不到,dogs<==>dog
    min_word_len        = 1 ## 最小索引词长度,小于这个长度的词不会被索引
    ngram_len       = 1 ## 简单分词,只支持0和1,如果要搜索中文,请指定为1
    ngram_chars     = U+3000..U+2FA1F ## 需要分词的字符,如果要搜索中文,请放开这行
    html_strip      = 0 ## html标记清理,是否从输出全文数据中去除HTML标记
}

indexer
{
    # memory limit, in bytes, kiloytes (16384K) or megabytes (256M)
    # optional, default is 128M, max is 2047M, recommended is 256M to 1024M
    mem_limit       = 128M ## 建立索引的时候,索引内存限制
}

searchd
{
    listen          = 9312 ## 监听端口
    listen          = 9306:mysql41

    log         = D:/sphinx/log/searchd.log ## 监听日志
    query_log       = D:/sphinx/log/query.log ## 查询日志
    pid_file        = D:/sphinx/log/searchd.pid ## ## 进程id文件
    read_timeout        = 5 ## 客户端读超时时间
    client_timeout      = 300 ## 客户端持久连接超时时间,即客户端读一次以后,持久连接,然后再读一次,中间这个持久连接的时间
    max_children        = 30 ## 并行执行搜索的数目
    persistent_connections_limit    = 30
    preopen_indexes     = 1 ## 索引预开启，是否强制重新打开所有索引文件
    unlink_old      = 1 ## 索引轮换成功之后，是否删除以.old为扩展名的索引拷贝
    max_packet_size     = 8M ## 网络通讯时允许的最大的包的大小
    max_filters     = 256 ## 每次查询允许设置的过滤器的最大个数
    max_filter_values   = 4096 ## 单个过滤器允许的值的最大个数
    max_batch_queries   = 32 ## 每次批量查询的查询数限制
    workers         = threads # for RT to work     多处理模式（MPM）。 可选项；可用值为none、fork、prefork，以及threads。 默认在Unix类系统为form，Windows系统为threads。

}
```
<br>
__注:配置文件更改完成之后,回到根目录新建配置文件中使用到的data和log两个目录__

导入源码里提供的示例数据,执行etc目录下的example.sql脚本:
![导入测试数据](https://upload-images.jianshu.io/upload_images/14618365-f72b58fee6c8b94f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

或者使用mysql图形处理界面工具:
![导入测试数据.png](https://upload-images.jianshu.io/upload_images/14618365-5e7913e3b77827cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 5. 根据sphinx.conf创建索引
```
## 构建索引语法:
indexer -c 配置文件 索引名字
```
 - 打开windows 命令提示行工具,进入到sphinx/bin目录下

![根据sphinx.conf创建索引](https://upload-images.jianshu.io/upload_images/14618365-b051e3066904d801.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 启动sphinx检索服务,后台常驻

![启动](https://upload-images.jianshu.io/upload_images/14618365-4f8b1e2d6cc27b6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 查看索引创建情况
重新打开一个命令行窗口,

![索引建立情况](https://upload-images.jianshu.io/upload_images/14618365-71ee147c00cedb1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 6. Sphinxapi.php使用(或者安装php扩展)
sphinx提供了各种语言的代码,php使用sphinx,只需要引入sphinxapi.php,打开根目录下的api目录,将sphinxapi.php文件复制到你的项目目录下,方便调用
```
<?php
/**
 * Demo.php
 * Create on 2018/8/29 10:42
 * Create by liu
 * Administrator
 */

class Demo extends CI_Controller
{
    private $sphinx;
    public function __construct()
    {
        parent::__construct();
        require_once APPPATH . 'libraries/Sphinxapi.php';
        $this->sphinx = new SphinxClient();
    }

    public function sphinx()
    {
        //设置操作哪个sphinx服务器
        $this->sphinx->setServer('localhost',9312);
        $keyword = "银行";//要搜索的关键字
        $index = 'bank';//索引名称
        //查询出关键字所在的主键id
        $this->sphinx->_limit = 2000;
        $res = $this->sphinx->Query($keyword,$index);
        echo '<pre>';

        if (isset($res['matches'])){
            $ids = array_keys($res['matches']);
            $ids = implode(',',$ids);
        }else{
            print_r("内容不存在");
            return;
        }
        //获取匹配到的主键id
        $mysql_con = mysqli_connect('localhost','root','','hongbao');//本地数据库
        mysqli_query($mysql_con,'set name utf8');
        mysqli_query($mysql_con,'use hongbao');
        $sql = "select * from bank where id in ($ids)";
        
        $res = mysqli_query($mysql_con,$sql);

        while ($row = mysqli_fetch_assoc($res)){
            $data[] = $row;
        }
        foreach ($data as $key => $v){

            $v = str_replace($keyword,"<font color='red'>{$keyword}</font>",$v);
            $data[$key] = $v;
        }
        print_r($data);

    }

}

```

代码运行结果
![代码运行结果](https://upload-images.jianshu.io/upload_images/14618365-4ce186161f4e1689.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)