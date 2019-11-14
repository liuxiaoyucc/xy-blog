---
title: PHP 单色位图取模
date: 2019-11-12 23:24:09
categories: 
    - [大熊BIGBEAR, PHP, GD]
    - [大熊BIGBEAR, 图像处理]
tags: 
    - PHP
    - GD
    - pctoLCD2002
    - bmp
    - 位图
    - 大熊BIGBEAR
---
<meta name="referrer" content="no-referrer" />

__2018-11-27日更新:__
&emsp;&emsp;由于没有找到生成`.bmp`格式图片的好办法,改为使用`.wbmp`格式,转换和读取都改为`.wbmp`格式,原来的`bmp2hex`函数逻辑没有变化,改名为`wbmp2hex`,并不再使用`ImageCreateFromBMP`函数,可以收藏一下这个函数还是有用的,最新的代码我也提供了下载在文章末尾

<!-- more -->


#### 准备阶段:
* __pctoLCD2002__
网上找到的一款取模软件,可以读取`.bmp`图片并生成字模,当然我们还是要用代码来完成,这个只是起到了一个对照作用,我将它放在了我的网盘下供大家下载
链接：[点我下载pctoLCD2002](https://pan.baidu.com/s/12X9Jbctz7wp8_wOWJCmYdw) 密码：`2lyl`

* __PHP GD扩展__
强大的PHP图像生成和处理扩展
* __Windows自带画图工具__
主要用来生成`.bpm`格式的图片,目前我还没有找到好的用PHP将`.jpg`和`.png`图片转为单色`.bmp`格式图片的办法,暂时只好用画图工具来生成


#### 操作步骤分解演示
##### 一. 使用画图工具获得.bmp格式图片
1. 使用画图工具打开一张事先准备好的图片,另存为`.bmp`__单色位图__,这样我们就得到了一张`.bmp`格式的图片,白色背景,只有黑色

![打开图片](https://upload-images.jianshu.io/upload_images/14618365-b6d0e5ed3d0f89b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![另存为](https://upload-images.jianshu.io/upload_images/14618365-036a928b957300bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![.bmp格式](https://upload-images.jianshu.io/upload_images/14618365-675d730990fe5b64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 或者我们自己动手来画一张,打开画图工具,调整画布大小为你需要的尺寸,示例为100*70像素,取消勾选保持纵横比,调整好后点击确定,然后我们可以用刷子随便画些什么在画布上,你喜欢就好,然后重复前面的__另存为.bmp单色位图__步骤

![image.png](https://upload-images.jianshu.io/upload_images/14618365-58921a2bee3a6c6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. pctoLCD2002也可以新建一幅.bmp图片,并且非常简单

![pctoLCD2002新建bmp文件](https://upload-images.jianshu.io/upload_images/14618365-d9a0e2dae4a0f54b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 二. 使用__`pctoLCD2002`__取模
找到PCtoLCD2002.exe并双击打开
###### 1. 规则解析,及本文配置项参考
在取字模之前我们先来说下PCtoLCD2002设置项和取模规则
* __配置信息:__

![pctoLCD2002设置项](https://upload-images.jianshu.io/upload_images/14618365-3e9e397592bf5c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* __取模说明:__
a. 逐行式逐列式:顾名思义就是读取每张图片时取点时是逐行还是逐列的
b. 取模走向:
__逆向__:从低位到高位
__顺向__:从高位到低位
举例:
`*`星号代表图中非空白的像素点,`_`代表空白的像素点,取八位为一个字节
`* _ _ _ _ _ _ _ `代表一个字节(为了方便查看,每个符号键我加入了一个空格,实际是没有的)
逆向即是从后往前写,表示为00000001
顺向即是从前往后写,表示为10000000
c. 输出数制:
这里选择十六进制,因需选择,不够我需要的是十六进制,后面的代码也只有十六进制的

* __本文取模规则:__
`逐行式` `顺向` ` 十六进制 `
从第一行开始,每行每隔8个像素点为一个字节,每行结尾最后不足8位,用0补满

###### 2. 生成字模
设置好规则后,打开之前制作的.bmp图片,点击生成字模,这时下方会生成出十六进制串,如图:

![image.png](https://upload-images.jianshu.io/upload_images/14618365-949d42ff637fec24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是这还不是我最后想要的格式,需要处理一下:
* 去掉开始处和结束处的文件路径
* 去掉所有的标点符号`,`和`'{' '}'`
* 去掉十六进制的标识部分,所有的`0x`

最后得到一串连贯的字符串,类似:
```
0000000000000007F8000000000000000000000000003FFC00000000000000000000000000000003FF80000000
```
这就是我们最终需要的部分了!下面我们用代码来实现这个功能:

#### 问题解决:
##### 一. 实现过程及思路
###### 0. 生成单色位图
卡在这里好久,钻进了死胡同,其实.wbmp的图片完全符合我的要求:
GD库就可以将`jpg/png`转换成`wbmp`格式,使用时可以调节`threshold`参数,解释如下,我理解为精度不知道准不准确,也没有查到阀值到底是什么意思...

![threshold](https://upload-images.jianshu.io/upload_images/14618365-a133d36f74a84e72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

生成`.wbmp`格式图片代码示例:
```
        $filename = 'static/img/1.jpg';
        $path = 'static/img/11.wbmp';
        $image = getimagesize($filename);
        jpeg2wbmp($filename, $path, $image[1], $image[0], 5);//threshold == 5时,和给的软件转换后结果完全一致
```

图片样式:

![.wbmp格式](https://upload-images.jianshu.io/upload_images/14618365-1db851391b978ca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 1. 读取.wbmp格式图片(原读取.bmp格式)
使用gd库的imagecreatefromwbmp函数:
```
        $filename = 'static/img/11.wbmp';
        $im = imagecreatefromwbmp($filename);
```
安装好GD库扩展后,我发现gd库只能读取.wbmp文件,并不支持.bmp文件,我的gd库版本信息如下`gd_info()`,经过一番google,找到了一个可以使用的读取.bmp的函数`ImageCreateFromBMP()`,感谢前辈

![gd_info()](https://upload-images.jianshu.io/upload_images/14618365-e4ddf9d0efeb1393.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 2. 逐个像素读取
可以读取.wbmp格式了,我们该如何能得出每个像素的颜色值呢?
通过查看gd库文档过程,我发现一个函数`imagecolorat()`,可以根据传入的位置,获取每个像素的索引值
使用示例:
```
        // 取得一点的颜色
        $file_name = '';//wbmp图片路径
        $im = imagecreatefromwbmp($file_name);//读取wbmp格式
        $start_x = 5;//行,从0开始
        $start_y = 10;//列,从0开始
        $color_index = imagecolorat($im, $start_x, $start_y);
        print_r($color_index);
```


###### 3. 获取图片宽高
第2步中可以获取每个像素中的值了,但是我们总不能每个点都手动传入,这时我们就需要获取图片的宽高了

gd库中有获取图片宽高的函数`imagesx()`和`imagesy()`,代码示例:
```
        $im = imagecreatefromwbmp($file_name);//读取bmp格式,非gd库
        $width = imagesx($im);
        $height = imagesy($im);
        echo $width.'*'.$height;
```
到这里,最主要的部分都已经可以获取到了,后面就是逻辑部分了(代码中可以看到具体实现方式):
* 根据图片的宽高,逐行逐点读取每个像素的值,每8位组合成1个字节,然后取模,再转为16进制
* 检测每一行的最后是否满足8个像素,不足则用0补满
* 最终将每一行组合到一起,组成16进制字符串

##### 二. 源码
__整个过程可以大概分为三步完成,你可以根据自己的需求参考或者直接copy使用,如果对你有帮助,希望可以点个赞,转载请注明本篇文章链接地址及作者,谢谢!__

* 将普通`jpg/png`格式转为`.wbmp`
```
        $filename = 'static/img/1.jpg';
        $path = 'static/img/11.wbmp';
        $image = getimagesize($filename);
        jpeg2wbmp($filename, $path, $image[1], $image[0], 5);//threshold == 5时,和给的软件转换后结果完全一致
```
* 读取`.wbmp`格式图片
```
        $filename = 'static/img/11.wbmp';
        $im = imagecreatefromwbmp($filename);
```
* 将`.wbmp`转为`hex_str`
```
/**
     * @param $im
     * @return string
     * Commented by liu
     */
    public function wbmp2hex($im)
    {
        $width = imagesx($im);
        $height = imagesy($im);
        $num = $width % 8;

        $hex_str = '';
        for ($start_y = 0; $start_y < $height; $start_y++) {
            $binary_str = '';
            for ($start_x = 0; $start_x < $width; $start_x++) {
                $color_index = imagecolorat($im, $start_x, $start_y);//指定像素的索引值

                $binary_str .= $color_index == 1 ? 1 : 0;
                if ((1 + $start_x) % 8 == 0 && $start_x != 0) {//每隔8位转换1次
                    $hex = (string)dechex(bindec($binary_str));
                    $hex = strlen($hex) == 1 ? '0' . $hex : $hex;//补0
                    $hex_str .= $hex;
                    $binary_str = '';
                }
            }

            //这时如果$binary_str不为空,说明需要向后补0
            if ($num) {
                for ($i = 0; $i < 8 - $num; $i++) {
                    $binary_str .= 0;
                }
                $hex = (string)dechex(bindec($binary_str));
                $hex = strlen($hex) == 1 ? '0' . $hex : $hex;//补0
                $hex_str .= $hex;
            }
        }

        $hex_str = strtoupper($hex_str);//转为大写
        return $hex_str;
    }
```


最后附上读取`.bmp`格式图片的函数:
```
<?php

/**
 * Commented by liu
 * Create on 2018/11/23 17:51
 * Class Image_api
 */

class Image_api
{
    function __construct (){

    }

    /**
     * ImageCreateFromBMP函数,读取bmp格式图片
     * 注:phpGD扩展中没有ImageCreateFromBMP函数,只有ImageCreateFromWBMP
     * @param $filename
     * @return bool|resource
     * Commented by liu
     */
    function ImageCreateFromBMP($filename)
    {
        //Ouverture du fichier en mode binaire
        if (!$f1 = fopen($filename, "rb"))
            return FALSE;

        //1 : Chargement des ent�tes FICHIER
        $FILE = unpack("vfile_type/Vfile_size/Vreserved/Vbitmap_offset", fread($f1, 14));
        if ($FILE['file_type'] != 19778)
            return FALSE;

        //2 : Chargement des ent�tes BMP
        $BMP = unpack('Vheader_size/Vwidth/Vheight/vplanes/vbits_per_pixel' .
            '/Vcompression/Vsize_bitmap/Vhoriz_resolution' .
            '/Vvert_resolution/Vcolors_used/Vcolors_important', fread($f1, 40));
        $BMP['colors'] = pow(2, $BMP['bits_per_pixel']);
        if ($BMP['size_bitmap'] == 0)
            $BMP['size_bitmap'] = $FILE['file_size'] - $FILE['bitmap_offset'];
        $BMP['bytes_per_pixel'] = $BMP['bits_per_pixel'] / 8;
        $BMP['bytes_per_pixel2'] = ceil($BMP['bytes_per_pixel']);
        $BMP['decal'] = ($BMP['width'] * $BMP['bytes_per_pixel'] / 4);
        $BMP['decal'] -= floor($BMP['width'] * $BMP['bytes_per_pixel'] / 4);
        $BMP['decal'] = 4 - (4 * $BMP['decal']);
        if ($BMP['decal'] == 4)
            $BMP['decal'] = 0;

        //3 : Chargement des couleurs de la palette
        $PALETTE = array();
        if ($BMP['colors'] < 16777216) {
            $PALETTE = unpack('V' . $BMP['colors'], fread($f1, $BMP['colors'] * 4));
        }

        //4 : Cr�ation de l'image
        $IMG = fread($f1, $BMP['size_bitmap']);
        $VIDE = chr(0);

        $res = imagecreatetruecolor($BMP['width'], $BMP['height']);
        $P = 0;
        $Y = $BMP['height'] - 1;
        while ($Y >= 0) {
            $X = 0;
            while ($X < $BMP['width']) {
                if ($BMP['bits_per_pixel'] == 24)
                    $COLOR = unpack("V", substr($IMG, $P, 3) . $VIDE);
                elseif ($BMP['bits_per_pixel'] == 16) {
                    $COLOR = unpack("n", substr($IMG, $P, 2));
                    $COLOR[1] = $PALETTE[$COLOR[1] + 1];
                } elseif ($BMP['bits_per_pixel'] == 8) {
                    $COLOR = unpack("n", $VIDE . substr($IMG, $P, 1));
                    $COLOR[1] = $PALETTE[$COLOR[1] + 1];
                } elseif ($BMP['bits_per_pixel'] == 4) {
                    $COLOR = unpack("n", $VIDE . substr($IMG, floor($P), 1));
                    if (($P * 2) % 2 == 0)
                        $COLOR[1] = ($COLOR[1] >> 4);
                    else
                        $COLOR[1] = ($COLOR[1] & 0x0F);
                    $COLOR[1] = $PALETTE[$COLOR[1] + 1];
                } elseif ($BMP['bits_per_pixel'] == 1) {
                    $COLOR = unpack("n", $VIDE . substr($IMG, floor($P), 1));
                    if (($P * 8) % 8 == 0)
                        $COLOR[1] = $COLOR[1] >> 7;
                    elseif (($P * 8) % 8 == 1)
                        $COLOR[1] = ($COLOR[1] & 0x40) >> 6;
                    elseif (($P * 8) % 8 == 2)
                        $COLOR[1] = ($COLOR[1] & 0x20) >> 5;
                    elseif (($P * 8) % 8 == 3)
                        $COLOR[1] = ($COLOR[1] & 0x10) >> 4;
                    elseif (($P * 8) % 8 == 4)
                        $COLOR[1] = ($COLOR[1] & 0x8) >> 3;
                    elseif (($P * 8) % 8 == 5)
                        $COLOR[1] = ($COLOR[1] & 0x4) >> 2;
                    elseif (($P * 8) % 8 == 6)
                        $COLOR[1] = ($COLOR[1] & 0x2) >> 1;
                    elseif (($P * 8) % 8 == 7)
                        $COLOR[1] = ($COLOR[1] & 0x1);
                    $COLOR[1] = $PALETTE[$COLOR[1] + 1];
                } else
                    return FALSE;
                imagesetpixel($res, $X, $Y, $COLOR[1]);
                $X++;
                $P += $BMP['bytes_per_pixel'];
            }
            $Y--;
            $P += $BMP['decal'];
        }

        //Fermeture du fichier
        fclose($f1);
        return $res;
    }

}
```
