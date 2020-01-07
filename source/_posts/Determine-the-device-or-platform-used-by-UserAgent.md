---
title: JS 通过User-Agent判断用户使用的设备 或平台
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
tags:
  - 前端
  - JavaScript
  - 大熊BIGBEAR
translate_title: js-determines-the-device-or-platform-used-by-user-through-useragent
date: 2019-11-12 22:08:21
---

<meta name="referrer" content="no-referrer" />

```
<script>
    var browser = {
        versions: function() {
            var u = navigator.userAgent,
                app = navigator.appVersion;
            return {
                trident: u.indexOf('Trident') > -1, //IE内核
                presto: u.indexOf('Presto') > -1, //opera内核
                webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核
                gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核
                mobile: !!u.match(/AppleWebKit.*Mobile.*/), //是否为移动终端
                ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
                android: u.indexOf('Android') > -1 || u.indexOf('Adr') > -1, //android终端
                iPhone: u.indexOf('iPhone') > -1, //是否为iPhone或者QQHD浏览器
                iPad: u.indexOf('iPad') > -1, //是否iPad
                webApp: u.indexOf('Safari') == -1, //是否web应该程序，没有头部与底部
                weixin: u.indexOf('MicroMessenger') > -1, //是否微信 （2015-01-22新增）
                qq: u.match(/\sQQ/i) == " qq", //是否QQ
                eleme: u.indexOf('Needle') > -1//是否饿了么
            };
        }(),
        language: (navigator.browserLanguage || navigator.language).toLowerCase()
    };
    console.log(browser['versions']['eleme']);
    if (browser['versions']['eleme']){
        show();
    }
</script>
```