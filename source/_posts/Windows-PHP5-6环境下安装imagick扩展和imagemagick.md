---
title: Windows+PHP5.6环境下安装imagick扩展和imagemagick
date: 2019-11-12 22:40:35
categories: 
    - [大熊BIGBEAR, PHP]
    - [大熊BIGBEAR, 图像处理]
tags:
    - PHP
    - imagick
    - Imagemagick
    - 大熊BIGBEAR
---


>其实回过头看,安装过程中最容易出错的反而是下载阶段,一定要将imagemagick和imagick的版本和`phpinfo`的信息对应好!
下图中几点需要注意,每个人的信息可能不同,根据你自己的phpinfo来选择接下来的下载的程序及扩展版本:
* PHP Version: __PHP版本__ 
* compiler: __MSVC11__
* Architecture: __x86__
* Thread Safety: __disabled__ 非线程安全,也就是**NTS**,相反的则是线程安全**TS**

<!-- more -->

![image.png](https://upload-images.jianshu.io/upload_images/14618365-bc2658ebe00f81fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


    
__正文部分__

*********


#### 一. Imagemagick部分
##### 1. 下载

* 下载`Imagemagick`程序
下载地址:[Imagemagick程序下载地址](http://windows.php.net/downloads/pecl/deps/)
打开链接,找到Imagemagick的下载区域,根据phpinfo我应该选择vc11,32位的下载链接,也就是下图中圈出的部分
![image.png](https://upload-images.jianshu.io/upload_images/14618365-1fa2dc2e9309896b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2. 安装
* **解压缩** 将下载后的压缩包,直接解压到你想放置的目录下,不要有中文和特殊字符,我把它放在了`D:\install_dir\imagemagick`下,并且记住不要忘记这个路径

* **环境变量配置** 为了更方便,全局都可以使用,我们需要配置下环境变量,进入刚才解压缩的目录,再进入/bin目录下,复制当前路径,我的路径为`D:\install_dir\imagemagick\bin`,右击我的电脑(计算机),按照下图依次打开环境变量配置位置,将路径粘贴进去(**注意:Path中可能有多个路径,多个路径间用`;`分号分割就好**)
![image.png](https://upload-images.jianshu.io/upload_images/14618365-b9d3e46e19449641.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **导入DLL文件** 进入`imagemagick`的`bin`目录下,复制所有`.dll`后缀的文件到你的`php`根目录下(我使用的是Phpstudy建的环境,我的`php`路径为:`D:\phpStudy\php\php-5.6.27-nts`)我的版本有147个文件,这里有个小技巧,在文件管理器右上角搜索`.dll`,然后全选复制,会方便一点
![image.png](https://upload-images.jianshu.io/upload_images/14618365-845b64c7cb6228f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 二. Imagick扩展部分
&emsp;&emsp;首先`imagick`一个`php`扩展,可以使用`php`控制`imagemagick`程序
##### 1. 下载
* **下载imagick** [下载地址](https://windows.php.net/downloads/pecl/releases/imagick/) 仍然是根据你的phpinfo选择对应的版本,根据文章开始的图片,我应该选择[php_imagick-3.4.3-5.6-ts-vc11-x86.zip](https://windows.php.net/downloads/pecl/releases/imagick/3.4.3/php_imagick-3.4.3-5.6-ts-vc11-x86.zip)

#####2. 安装
* **php_imagick.dll** 找到刚下载的压缩包,解压后,找到`php_imagick.dll`文件,将其复制粘贴到`php`根目录下的`ext`目录下
* **其他`.DLL`文件** 将解压后的`imagick`目录下的其他`.dll`后缀的文件全部复制粘贴到`php`根目录下
* **php.ini** 找到php.ini文件并打开编辑,加入`extension=php_imagick.dll`这一行

#### 三. 重启
到这里基本上可以成功安装了,`imagemagick`需要重启电脑后才会生效,重启电脑后,查看phpinfo,如果成功安装了,会看到下图中的部分
![image.png](https://upload-images.jianshu.io/upload_images/14618365-e71f02e2d9ef466b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
