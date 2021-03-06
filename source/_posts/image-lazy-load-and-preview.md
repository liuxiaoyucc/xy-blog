---
title: 图片懒加载 滚动加载 点击图片预览实现过程
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
tags:
  - 前端
  - JavaScript
  - WeUI
  - Jquery WEUI
  - ECHO JS
  - 懒加载
  - 大熊BIGBEAR
translate_title: image-lazy-loading-scroll-click-preview-implementation-process
date: 2019-11-12 22:03:12
---
<meta name="referrer" content="no-referrer" />

>作者是个前端菜鸟,只能靠着东拼西凑才能生存下来这样子

上次写了一个实现__搜索框自动补全__的小功能的文章,今天这个在其基础上,加入了几个新功能,两者卡可以结合使用,也可以分开独自使用,没有影响的,想了解的同学,可以先去了解一下,__[[传送门]](https://www.jianshu.com/p/18047be090f4)__,下面开始实现标题的功能

#### 1. 准备阶段

* __WEUI__: WEUI是一套同微信原生视觉体验一致的基础样式库,由微信官方设计团队为微信内网页和小程序量身定制,令用户感知更加统一,[点击进行在线体验](https://weui.io/).

* __Jquery WEUI__:JQuery WeUI 是专为微信公众账号开发而设计的一个简洁而强大的UI库，包含全部WeUI官方的CSS组件，并且额外提供了大量的拓展组件，丰富的组件库可以极大减少前端开发时间,[JQuery WeUI官网](http://jqweui.com/).

* __ECHO JS__:ECHO JS 是一个纯javascript轻量级延迟加载插件,用来实现懒加载部分

<!-- more -->


#### 2. 实现思路
* __监听键盘__:监听键盘的搜索动作(也就是Enter键), 接收到后台的数据后,循环append到img容器中,这里使用的是`WEUI`的九宫格,将三列调成了两列,将图片展示出来

* __点击预览__ 监听用户的鼠标点击动作,当用户点击某一张图片时,调用[Photo Browser](http://jqweui.com/extends#photos),并使用```pb.open();```打开预览图

* __懒加载__ 将需要懒加载的img标签添加`data-echo`属性,并且将原src改为一张透明的loading gif图,echojs就会自动实现懒加载了,使用起来非常简单

* __滚动加载__ 滚动加载使用的是JQuery WEUI的[infinite](http://jqweui.com/extends#infinite),当滑动到最下面时,触发加载动作,向后台发起ajax请求,同时记录当前页数

#### 3. 不多B,上代码
```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>图片搜索</title>
    <link rel="stylesheet" href="/static/jquery-weui-build/dist/lib/weui.min.css">
    <link rel="stylesheet" href="/static/jquery-weui-build/dist/css/jquery-weui.css">
    <link rel="stylesheet" href="/static/jquery-weui-build/demos/css/demos.css">
    <link rel="stylesheet" href="/static/jquery-ui/jquery-ui.css">


    <script src="/static/js/jquery-3.3.1.min.js"></script>
    <script src="/static/jquery-weui-build/dist/js/jquery-weui.js"></script>
    <script type='text/javascript' src='/static/jquery-weui-build/dist/js/swiper.js' charset='utf-8'></script>
    <script src="/static/jquery-ui/jquery-ui.js"></script>
    <script src="/static/waterfall/js/echo.min.js"></script>
    <script>
        echo.init({//初始化echo.js
            offset: 0,
            throttle: 0
        });
    </script>
    <style>
        body {
            background: #efeff4;
        }

        img {
            width: 100%;
            height: 100%;
        }

    </style>
</head>
<body>
<div id="body">
    <div class="ui-widget">
        <div class="weui-search-bar" id="searchBar">
            <form class="weui-search-bar__form" action="#">
                <div class="weui-search-bar__box">
                    <i class="weui-icon-search"></i>
                    <input type="search" class="weui-search-bar__input" id="tags" placeholder="搜索" required="">
                    <a href="javascript:" class="weui-icon-clear" id="searchClear"></a>
                </div>
                <label class="weui-search-bar__label" id="searchText"
                       style="transform-origin: 0px 0px 0px; opacity: 1; transform: scale(1, 1);">
                    <i class="weui-icon-search"></i>
                    <span>搜索</span>
                </label>
            </form>
            <a href="javascript:" class="weui-search-bar__cancel-btn" id="searchCancel">取消</a>
        </div>
    </div>
    <div class="weui-grids" id="item"></div>
    <div class="weui-loadmore">
        <i class="weui-loading"></i>
        <span class="weui-loadmore__tips">正在加载</span>
    </div>
</div>

<script>
    var availableTags = [];//数据源
    var keyword = '';//搜索关键字
    var page = 1;//当前页数


    //先初始化自动补全功能
    $("#tags").autocomplete({
        source: availableTags //数据源
    });

    if (!$('#item').html()){
        $('.weui-loadmore').html('<div class="weui-loadmore weui-loadmore_line"> <span class="weui-loadmore__tips">暂无数据</span> </div>');//样式需要调整
    }

    //去掉字符串中任意位置的空格
    function Trim(str, is_global) {
        var result;
        result = str.replace(/(^\s+)|(\s+$)/g, "");
        if (is_global.toLowerCase() == "g") {
            result = result.replace(/\s/g, "");
        }
        return result;
    }

    //判断字符串是否全是中文
    function isChn(str) {
        var reg = /^[\u4E00-\u9FA5]+$/;
        if (!reg.test(str)) {
            return false;
        } else {
            return true;
        }
    }

    //请求后端获取数据源
    function get_source(word = null) {
        var url = "<?php echo base_url('Picture/source');?>?keyword=" + word;
        $.get({
            type: 'GET',
            url: url,
            async: false,//改为同步
            dataType: 'json',
            success: function (response) {
                availableTags = response;
            }
        });
    }
    //搜索和自动补全结合
    $('#tags').keyup(function () {
        keyword = $('#tags').val();
        if (event.which == '13') {
            $('#item').empty();//初始化图片列表
            page = 1;//初始化当前页
            call();
            return;
        }
        if (isChn(Trim(keyword, 'g'))) {
            get_source(keyword);
            $("#tags").autocomplete({
                source: availableTags //数据源
            });
        }
    });

    //向后台请求数据
    function call() {
        var pay_url = "<?php echo base_url('Picture/search');?>";
        $.ajax({
            type: 'GET',
            url: pay_url + '?keyword=' + keyword + '&page=' + page, //搜索
            dataType: 'json',
            success: function (data) {

                if (data['errno'] == 40001){
                    no_data_style();
                    return;
                }
                $('.weui-loadmore').html('<i class="weui-loading"></i> <span class="weui-loadmore__tips">正在加载</span>');
                $.each(data, function (index, item) {
                    $('#item').append('<a href="javascript:void(0)" class="weui-grid" style="width: 50%;" onclick=big_img("'+item.img+'","'+Trim(item.name, 'g')+'")> <img src="/static/img/index_32.png" data-echo="'+item.img+'" alt="'+item.name+'"> <p style="text-align: center;font-size: 12px;color: black;margin-top: 10px">'+item.name+'</p></a>');
                    if (data.length <= 8 && page == 1){
                        no_data_style();
                    }
                });
                echo.init({//获得数据后初始化echojs
                    offset: 0,
                    throttle: 0
                });
            }
        })
    }

    function no_data_style() {
        $('.weui-loadmore').empty();
        $('.weui-loadmore').append('<div class="weui-loadmore weui-loadmore_line"> <span class="weui-loadmore__tips">暂无更多</span> </div>');//样式需要调整
    }
</script>

<!--点击预览全图-->
<script>
    function big_img(img,name) {
        var imgs = [{'image':img,'caption':name}];
        var pb = $.photoBrowser({
            items: imgs,
        });
        pb.open();
    }
</script>

<!--滚动加载-->
<script>
    //滚动加载
    var loading = false;
    $(document.body).infinite().on("infinite", function () {
        if (loading) return;
        loading = true;
        setTimeout(function () {
            page++;
            call();
            loading = false;
            echo.init({
                offset: 0,
                throttle: 0
            });
        }, 1000);
    });
</script>

</body>
</html>

```