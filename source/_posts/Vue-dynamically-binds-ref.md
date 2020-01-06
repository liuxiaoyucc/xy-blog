---
title: Vue动态绑定ref
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
    - Vue
    - uni-app
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
    - Vue
    - WEEX
tags:
  - Js
  - Vue
  - uni-app
  - WEEX
  - 随手记
toc: true
translate_title: vue-dynamic-binding-ref
date: 2019-11-13 09:54:18
---
<!-- <meta name="referrer" content="no-referrer" /> -->

>记录一下动态绑定及使用ref的方法
<!-- more -->

## 情景
__需要在使用weex的`<list>`组件时,需要实现一个滚动到具体某一个`<cell>`的功能__

## 实现
### 在for循环标签上动态绑定ref
{% codeblock lang:html %}
<div v-for="(item, index) in list" :key="index" :ref="`item${index}`"></div>
{% endcodeblock %}

### 获取ref并滚动到指定位置
#### 引入weex的dom 模块

{% codeblock lang:javascript %}
const dom = weex.requireModule('dom');
{% endcodeblock %}
#### 动态获取ref,并使用scrollToElement滚动到指定元素
{% codeblock lang:javascript %}
const el = this.$refs[`message${length-1}`][0];
dom.scrollToElement(el, {});
{% endcodeblock %}