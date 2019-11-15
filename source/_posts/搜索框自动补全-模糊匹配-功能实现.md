---
title: 搜索框自动补全(模糊匹配)功能实现
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
    - JQuery UI
    - autocomplete
tags:
  - 前端
  - JavaScript
  - JQuery
  - 大熊BIGBEAR
translate_title: search-box-auto-completion-fuzzy-matching-function-implementation
date: 2019-11-12 21:57:06
---
<!-- <meta name="referrer" content="no-referrer" /> -->


&emsp;&emsp;本地实现了一个搜索框自动补全的小功能,在JQuery UI的autocomplete插件的基础上,加入了自己的业务代码,贴出来回顾一下,同时可以给大家一个参考

<!-- more -->

&emsp;&emsp;首先贴出的是JQuery Ui 的自动补全插件部分的代码,后面的功能都是在其基础上追加的,直接拷贝到你的本地就可以直观的看到运行效果,也可以到官网上面体验和查看,为了方便,我这里是直接引入的JS链接[点击下载JQuery UI的源码](https://jqueryui.com/download/)
```
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>jQuery UI Autocomplete - Default functionality</title>
  <link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">
  <link rel="stylesheet" href="/resources/demos/style.css">
  <script src="https://code.jquery.com/jquery-1.12.4.js"></script>
  <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
  <script>
  $( function() {
    var availableTags = [//这里要改成根据用户的输入,自动更换词库的形式
      "ActionScript",
      "AppleScript",
      "Asp",
      "BASIC",
      "C",
      "C++",
      "Clojure",
      "COBOL",
      "ColdFusion",
      "Erlang",
      "Fortran",
      "Groovy",
      "Haskell",
      "Java",
      "JavaScript",
      "Lisp",
      "Perl",
      "PHP",
      "Python",
      "Ruby",
      "Scala",
      "Scheme"
    ];
    $( "#tags" ).autocomplete({//调用补全功能
      source: availableTags
    });
  } );
  </script>
</head>
<body>
 
<div class="ui-widget">
  <label for="tags">Tags: </label>
  <input id="tags">
</div>
 
 
</body>
</html>
```
运行截图

![jquery-ui的自动补全功能截图](https://upload-images.jianshu.io/upload_images/14618365-228044e216af0634.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


__下面说下我追加的部分功能及实现思路,有可以改进的地方还请指教:__

1. 首先,数据源要根据用户输入的内容实时更新.
输入框的值随着用户的输入会一直变动.所以,输入框下方的推荐补全的内容要输入的值进行变动,这里使用onkeyup属性来监听键盘动作,并传递此时input的value值到js函数中.
```
    //html
    <input type="search" class="" id="tags" placeholder="搜索" required="" onkeyup="catch_keyword(this.value)">

    //js代码
    function catch_keyword(word) {//这里接受并log出value
        console.log(word);
    }
```
2. 第2步,考虑到数据库中需要模糊检索的字段都是中文的菜品名称.所以,当用户输入字母的时候,进行了一下过滤,当输入的内容中存在字母时,不提交给后台处理
```
    //字符串判断函数
    //判断一个字符串是否混有字母,全中文返回true
    function isChn(str) {
        var reg = /^[\u4E00-\u9FA5]+$/;
        if (!reg.test(str)) {
            return false;
        } else {
            return true;
        }
    }
```

3. 发现当字符串中含有空格的时候,上面的字符串判断函数,返回的内容不符合预期,然后加入了一个去除字符串中所有空格的功能
```
    //去掉字符串中任意位置的空格,返回去除空格后的字符串
    function Trim(str, is_global) {
        var result;
        result = str.replace(/(^\s+)|(\s+$)/g, "");
        if (is_global.toLowerCase() == "g") {
            result = result.replace(/\s/g, "");
        }
        return result;
    }
```

4. 处理结束用户的输入后,就是提交到后台,然后返回数据源了,也就是availableTags;这里我把availableTags声明为全局变量.并且用同步的Ajax方式取回数据,然后赋值给availableTags,然后在监听键盘的函数中,使用返回的数据调用自动补全功能.
```
    //请求后端获取数据源
    function get_source(word = null) {
        var url = "<?php echo base_url('admin/Demo/source');?>?keyword=" + word;
        $.get({
            type: 'GET',
            url: url,
            async: false,//改为同步
            dataType: 'json',
            success: function (response) {
                console.log('1');
                availableTags = response;
            }
        });
    }

```

&emsp;这里更新下最开始的接收监听键盘后的value值的函数

```
    //捕捉键入的关键字
    function catch_keyword(word = null) {
        if (isChn(Trim(word, 'g'))) {//去掉空格后检查字符串,如果符合,继续请求后台
            get_source(word);
            $("#tags").autocomplete({
                source: availableTags //数据源
            });
        }
    }
```

5. 到这里,这个功能已经基本结束了,在测试过程中发现了一个小问题,__每次第一次获取用户输入的时候,自动补全功能没有触发,在用户继续输入后,才触发成功__,经过调试,我在页面加载完成后,初始化一下自动补全插件,解决了这个问题

__附: 完整代码__
不知道如何在markdown中添加下载链接,只好把完整代码放上来啦!
```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title></title>
    <link rel="stylesheet" href="/jquery-weui-build/dist/lib/weui.min.css">
    <link rel="stylesheet" href="/jquery-weui-build/dist/css/jquery-weui.css">
    <link rel="stylesheet" href="/jquery-weui-build/demos/css/demos.css">
    <link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">

    <script src="https://code.jquery.com/jquery-1.12.4.js"></script>
    <script src="/static/jquery-weui-build/dist/lib/fastclick.js"></script>
    <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>


    <script>
        $(function () {
            FastClick.attach(document.body);
        });
    </script>
    <script src="/jquery-weui-build/dist/js/jquery-weui.js"></script>
</head>
<body>

<div class="ui-widget">


    <div class="weui-search-bar" id="searchBar">
        <form class="weui-search-bar__form" action="#">
            <div class="weui-search-bar__box">
                <i class="weui-icon-search"></i>
                <input type="search" class="weui-search-bar__input" id="tags" placeholder="搜索" required=""
                       onkeyup="catch_keyword(this.value)">
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

<script>
    var availableTags = [];//数据源

    //先初始化自动补全功能
    $("#tags").autocomplete({
        source: availableTags //数据源
    });

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

    //捕捉键入的关键字
    function catch_keyword(word = null) {

        if (isChn(Trim(word, 'g'))) {
            get_source(word);
            $("#tags").autocomplete({
                source: availableTags //数据源
            });

        }
    }

    //请求后端获取数据源
    function get_source(word = null) {
        var url = "<?php echo base_url('admin/Demo/source');?>?keyword=" + word;
        $.get({
            type: 'GET',
            url: url,
            async: false,//改为同步
            dataType: 'json',
            success: function (response) {
                console.log('1');
                availableTags = response;
            }
        });
    }

</script>
</body>
</html>

```