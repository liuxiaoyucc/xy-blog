---
title: ThinkPHP框架搭建
date: 2019-11-12 23:38:23
categories: 
    - [大熊BIGBEAR, PHP, ThinkPHP]
tags:
    - PHP
    - ThinkPHP
    - 大熊BIGBEAR
---

`thinkphp`基于__`MVC`__的方式来组织,MVC是一种经典的程序设计理念,此模式将你的项目分成三个部分,`模型层(model)`,`视图层(view)`,`控制层(controller),MVC为这三个三次的首字母缩写



1. 下载ThinkPHP框架
官网下载:http://www.thinkphp.cn/
thinkphp有核心包和完整包之分,我这里选择的是最新版5.0.21的完整版
__* 核心包__:仅包含thinkphp运行的最主要文件,不包含扩展类,示例,文档
__* 完整包__:核心包的基础上增加了扩展类,示例及文档

下载后将文件解压到你的localhost根目录下,我的路径是:`D:\phpStudy\WWW`,然后重命名目录名为myphpstudy,方便我们使用
![重命名](https://upload-images.jianshu.io/upload_images/14618365-5ed1a4c8d854e68b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来在浏览器地址栏输入:`http://localhost/mythinkphp/public/`,显示出如下内容,说明你的thinkphp框架可以正常使用了.


2.目录结构

```
project  应用部署目录
├─application           应用目录（可设置）
│  ├─common             公共模块目录（可更改）
│  ├─index              模块目录(可更改)
│  │  ├─config.php      模块配置文件
│  │  ├─common.php      模块函数文件
│  │  ├─controller      控制器目录
│  │  ├─model           模型目录
│  │  ├─view            视图目录
│  │  └─ ...            更多类库目录
│  ├─command.php        命令行工具配置文件
│  ├─common.php         应用公共（函数）文件
│  ├─config.php         应用（公共）配置文件
│  ├─database.php       数据库配置文件
│  ├─tags.php           应用行为扩展定义文件
│  └─route.php          路由配置文件
├─extend                扩展类库目录（可定义）
├─public                WEB 部署目录（对外访问目录）
│  ├─static             静态资源存放目录(css,js,image)
│  ├─index.php          应用入口文件
│  ├─router.php         快速测试文件
│  └─.htaccess          用于 apache 的重写
├─runtime               应用的运行时目录（可写，可设置）
├─vendor                第三方类库目录（Composer）
├─thinkphp              框架系统目录
│  ├─lang               语言包目录
│  ├─library            框架核心类库目录
│  │  ├─think           Think 类库包目录
│  │  └─traits          系统 Traits 目录
│  ├─tpl                系统模板目录
│  ├─.htaccess          用于 apache 的重写
│  ├─.travis.yml        CI 定义文件
│  ├─base.php           基础定义文件
│  ├─composer.json      composer 定义文件
│  ├─console.php        控制台入口文件
│  ├─convention.php     惯例配置文件
│  ├─helper.php         助手函数文件（可选）
│  ├─LICENSE.txt        授权说明文件
│  ├─phpunit.xml        单元测试配置文件
│  ├─README.md          README 文件
│  └─start.php          框架引导文件
├─build.php             自动生成定义文件（参考）
├─composer.json         composer 定义文件
├─LICENSE.txt           授权说明文件
├─README.md             README 文件
├─think                 命令行入口文件
```

3. Hello World
这时你可以将框架目录导入到你的IDE中了,方便我们编辑代码,下面我们来试着输出Hello World 吧!
找到并打开如图所示的文件位置,将其中代码进行修改
![入口](https://upload-images.jianshu.io/upload_images/14618365-5505dd971cf24493.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改后的代码
```
<?php
namespace app\index\controller;

class Index
{
    public function index()
    {
        return "Hello World";
    }
}
```
这时我们去浏览器地址栏输入`http://localhost/mythinkphp/public/`就可以看到我们改动后的内容了,是不是很简单呢!
![Hello World](https://upload-images.jianshu.io/upload_images/14618365-4d4be39dc00c9209.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
