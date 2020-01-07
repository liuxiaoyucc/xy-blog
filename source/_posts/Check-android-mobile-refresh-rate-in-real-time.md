---
title: 查看Andriod手机实时刷新率
categories:
  - - 大熊BIGBEAR
    - 杂
tags:
  - adb
  - Andriod
  - 大熊BIGBEAR
translate_title: view-realtime-refresh-rate-of-andriod-phones
date: 2019-11-12 21:53:56
---
<meta name="referrer" content="no-referrer" />

#### 1. 下载ADB
#### 2. `adb shell "dumpsys window|grep mCurrentFocus`  获取app包名
#### 3. 操作需测试的app
#### 4. adb shell dumpsys gfxinfo 'app的包名'  >FPS.txt  输出日志
#### 5. 打开查看`Profile data in ms`下的内容:
`FPS = 1000/Draw + Prepare + Process + Execute`


![1551852181(1).jpg](https://upload-images.jianshu.io/upload_images/14618365-1b54255c3ec7b157.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->


