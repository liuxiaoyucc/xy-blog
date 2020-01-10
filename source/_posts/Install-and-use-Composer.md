---
title: Composer的安装及使用
date: 2020-01-08 23:00:03
categories:
  - - 大熊BIGBEAR
    - PHP
    - Composer
tags:
  - PHP
  - Composer
---

<meta name="referrer" content="no-referrer" />

> Composer 是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件

<p align="center">
    <a href="https://www.phpcomposer.com/" target="_blank" rel="noopener noreferrer">
        <img width="400" src="https://www.phpcomposer.com//assets/img/phpcomposer.png" alt="Composer logo">
    </a>
</p>

或许你还没听说过Composer,  亦或听说过 ~~没听过,两万五千里~~ 而没有使用过, 没关系, 跟着我一起, 用起来

<!-- more -->

## 网站

* [Composer官网](https://getcomposer.org/)
* [Composer中文网](https://www.phpcomposer.com/)

## 环境

* PHP

这是必须的吧, 啊哈哈哈, 如果还没有安装php, 我建议你先不要看下去了

## 安装

随便打开一个上面提供的网址, 找到下载页, 或者直接点击[Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)开始下载exe文件

双击Composer-Setup.exe开始安装

1. 这里勾上developer mode可以自定义安装目录, 也可以不勾选, Next



![composer-setup](https://user-images.githubusercontent.com/33248133/71993077-742cae80-3271-11ea-8ecb-338cc7bcb5cd.PNG)



2. 这里会自动识别出你的php安装目录, 如果没有识别出来, 需要自己选择一下, 所以要记得你的php装在什么位置了哦 Next



![composer-setup](https://user-images.githubusercontent.com/33248133/71993490-26fd0c80-3272-11ea-950c-c9a6ee791e12.PNG)



3. 设置代理, 按需设置, 没有可以直接跳过 Next



![composer-setup](https://user-images.githubusercontent.com/33248133/71993796-aab6f900-3272-11ea-86f7-e4bf17735152.PNG)



4. 这里来检查一下安装路径, 并且会自动帮你将composer加入到环境变量中, 这样就可以在全局使用了,填写无误可以点击 Install



![composer-setup](https://user-images.githubusercontent.com/33248133/71993953-e81b8680-3272-11ea-9276-7a4f29ace4ca.PNG)



5. 耐心等待一小会, 等他安装完成, 跳出警告可以直接点击下一步, 看到下图, 就已经安装完成啦! Next->Finished



![composer-five](https://user-images.githubusercontent.com/33248133/71994302-73951780-3273-11ea-8a91-496296ad9aac.PNG)



6. 打开命令行, 输入`composer -V`, 会看到相应的版本信息. 命令行需要重新打开哦, 如果没有显示出相应信息, 则需要重启一下电脑, 至此, composer就安装完成了



![composer-v](https://user-images.githubusercontent.com/33248133/71994695-2c5b5680-3274-11ea-98e7-4a4bc1d532a4.PNG)



## 使用

我们先从一个空项目说起

随便在哪里新建一个空的目录

### 安装依赖

#### 从composer.json开始

##### 新建composer.json文件

新建composer.json文件,输入下面内容, [`querylist`](http://www.querylist.cc/)是一个php的内容采集框架,后面我会写一篇关于它的使用方式, 这里我们使用它来演示composer

```json
{
	"require": {
		"jaeger/querylist": "4.0.*"
	}
}
```

##### composer install

在项目目录下执行`composer install` , composer会读取composer.json中的内容,来自动下载安装相关依赖

![演示动图](https://user-images.githubusercontent.com/33248133/71997841-7eeb4180-3279-11ea-8356-eb892fa78d22.gif)



##### 结束

等待安装完成后我们会在项目根目录下看到一个vendor目录, 和一个composer.lock文件



#### 从composer require开始

##### 命令

```php
composer require jaeger/querylist 4.0.*
```

##### 说明

使用这种方式, composer会自动生成一个composer.json文件, 并将jaeger/querylist写入到其中, 然后根据composer.json下载相关依赖, 命令执行完毕后, 项目根目录下的结构, 和前面的方式是一样的



#### 锁文件`composer.lock`

##### 生成

每次安装新的依赖时, Composer会将其相关信息, 自动写入lock文件中, 包括对应版本

##### 作用

如果多人协同项目的话, 其他人在执行composer install时, 会先从lock文件中寻找该依赖和相应版本, 保证了所有人的版本一致, 可以理解为lock文件, '__锁定__' 了各依赖的版本

##### 更新依赖版本

注意, 当执行__`composer update`__ 时,  composer会获取依赖的最新版本, 并更新lock文件, 如果你只想更新特定的某个包, 那么可以在命令后指明要更新的包,像这样

```json
composer update jaeger/querylist # 后面可以继续添加多个, 会一并更新
```

下面是lock中, 我们刚刚装好的依赖的部分信息, 你可以自己动手打开lock文件看下全部



![composer-lock中的部分依赖信息](https://user-images.githubusercontent.com/33248133/72072870-c3361a80-3329-11ea-96d3-b269196385eb.PNG)



### 引入



#### autoload.php

vendor目录生成后, Composer会帮我们准备一个自动加载文件, __autoload.php__, 我们只需引入这个文件, Composer会帮我们处理一切

<img src="https://user-images.githubusercontent.com/33248133/72047325-3ffad180-32f5-11ea-8ce3-c0c5e536275b.png" alt="autoload.php位置" style="zoom:80%;" />





#### 普通项目引入

##### 入口文件

我们在刚才的项目目录下新建一个可执行php文件, 命名为Index.php,

##### 代码实现

写入如下代码

这段代码抓取了我的个人博客首页的文章title和链接, 后面会有专门一篇文章介绍QueryList的使用, 很方便



```php
<?php
require 'vendor/autoload.php';
Use QL\QueryList;

$url = 'http://blog.pulsating.cn';
$rules = [
	'title'=> ['.article-title', 'text'],
	'url'=> ['.article-title', 'href']
];

$result = QueryList::rules($rules)->get($url)->queryData();

print_r($result);
```



执行Index.php, 可以看到已经成功的使用了querylist依赖



![执行结果](https://user-images.githubusercontent.com/33248133/72077886-ff21ad80-3332-11ea-970b-daf9b7442fe6.png)



#### 在CodeIgniter中引入

##### 修改config.php

将`$config['composer_autoload']`的值改为true, CI会自动在application/vendor/目录下寻找autoload.php文件并加载

```php
// application/config/config.php

$config['composer_autoload'] = TRUE;
```

或者也可以这样, 将值写成具体路径

```php
$config['composer_autoload'] = 'vendor/autoload.php';
```



##### controller中使用

直接在controller中的class外部use, 然后就可以使用QueryList了, 后续代码和上面相同

```php
use QL\QueryList;
```



## 结束

* 不推荐手写composer.json文件, 推荐使用require的方式下载依赖, 手写很容易出错



* [packagist](https://packagist.org/) 是 Composer 主要的一个包信息存储库, 你可以分享自己的package到上面, 也可以在里面找其他人分享的package使用



* [QueryList](http://www.querylist.cc/) 文中的QueryList官网