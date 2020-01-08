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
> Composer 是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

<p align="center">
    <a href="https://www.phpcomposer.com/" target="_blank" rel="noopener noreferrer">
        <img width="400" src="https://www.phpcomposer.com//assets/img/phpcomposer.png" alt="Composer logo">
    </a>
</p>

或许你还没听说过Composer,  亦或听说过 ~~没听过,两万五千里~~ 而没有使用过, 没关系, 跟着我一起, 用起来

<!-- more -->

## 网站

* 官网: [Composer](https://getcomposer.org/)
* 中文网: [Composer中文网](https://www.phpcomposer.com/)

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

#### 从composer.json开始



1. 新建一个空的项目目录
2. 新建composer.json文件,输入下面内容, [`querylist`](http://www.querylist.cc/)是一个php的内容采集框架,后面我会写一篇关于它的使用方式, 这里我们使用它来演示composer

```json
{
	"require": {
		"jaeger/querylist": "4.0.*"
	}
}
```

3. 在项目目录下执行`composer install` , composer会读取composer.json中的内容,来自动下载安装相关依赖

![2020-01-09-00-39-02](https://user-images.githubusercontent.com/33248133/71997841-7eeb4180-3279-11ea-8356-eb892fa78d22.gif)



4. 等待安装完成后我们会在项目根目录下看到一个vendor目录, 和一个composer.lock文件



#### 从composer require开始



直接执行命令

```php
composer require jaeger/querylist 4.0.*
```

使用这种方式, composer会自动生成一个composer.json文件, 并将jaeger/querylist写入到其中, 然后根据composer.json下载相关依赖, 命令执行完毕后, 项目根目录下的结构, 和前面的方式是一样的

