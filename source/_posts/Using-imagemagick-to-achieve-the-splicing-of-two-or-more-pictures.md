---
title: 使用imagemagick实现两张或多张图片图片拼接 覆盖
categories:
  - - 大熊BIGBEAR
    - PHP
  - - 大熊BIGBEAR
    - 图像处理
tags:
  - PHP
  - imagick
  - Imagemagick
  - 大熊BIGBEAR
translate_title: use-imagemagick-to-achieve-two-or-more-image-stitching
date: 2019-11-12 23:40:50
---

<meta name="referrer" content="no-referrer" />

>`Imagemagick`是一个强大的图像处理库,号称命令行上的photoshop

* [使用GD扩展实现和本篇文章相同的效果](http://www.pulsating.cn/2019/11/15/Use-GD-library-to-implement-image-merge/)
* [php的imagick扩展和imagemagick的安装教程](http://pulsating.cn/2019/11/12/install-imagick-extend-and-imagemagick-in-windows-php5.6/)


##### 1. 问题场景
&emsp;&emsp;在进行手上的一个海报项目时,遇到了这样一个需求:
&emsp;&emsp;如图,在用户制作一张海报后,最后保存的时候,图片主体`body`是用户制作的海报,`footer`是由左右两张二维码组成的,左边为公众号的带参数二维码,右侧是用户自定义的二维码,美工提供一张通用的footer模版,程序来负责动态将二维码替换上去,最终拼接成下面图片示例的样子
<!-- more -->

![海报示例](https://upload-images.jianshu.io/upload_images/14618365-5507c013705956a1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 2. 为什么使用imagemagick
&emsp;&emsp;整个项目后台使用的是PHP语言,既然涉及到图片处理,第一个想到的肯定是大名鼎鼎的`GD`库扩展,在使用gd库尝试后,因为性能的原因,可能是我使用的方式不对,合成一张图片的时间长的忍受不了,又果断放弃了,后面又想到使用`imagemagick`这个工具,试着使用iamgemagick来重写一遍,果然,速度快了许多

##### 3. 实现思路

&emsp;&emsp;a. 将body和footer上下拼接起来
&emsp;&emsp;b. 分别将获取的两个二维码图片根据坐标置入左下和右下
__注:我觉得先进行b步骤,再拼接,效率可能会高一丢丢,但是我也懒得测了,就没有改动__

##### 4. 代码部分
直接使用的是imagemagick的命令行指令,没有使用imagick扩展,已上线稳定使用
```
public function merge_img($file_name, $ad_file_name, $user_qrcode,$template_id)
    {
        $body = $file_name;//这个是body的路径
        $footer = APPPATH . '../static/img/footer.png';//footer是一张模板图
        $qrcode = APPPATH . '../' . $user_qrcode;//右下角用户自定义的二维码
        $param_qrcode = $this->param_qrcode($template_id);//左下角的带参数二维码,param_qrcode()方法返回其路径

        $cmd = 'convert -append '.$header.' '.$footer.' '.$ad_file_name.'';//1.将body和footer拼接,因为body和footer是固定等宽的,所以直接拼接,+append为横向拼接,-append为纵向拼接
        $result = exec($cmd);//执行

        $cmd = 'composite -geometry +634+1093 '.$qrcode.' '.$ad_file_name.' '.$ad_file_name.'';//将用户二维码置入右下角,'+634+1093'为置入位置的坐标
        $result = exec($cmd);

        $cmd = 'composite -geometry +27+1093 '.$param_qrcode.' '.$ad_file_name.' '.$ad_file_name.'';//将带参数二维码置入左下角
        $result = exec($cmd);
    }
```
