---
title: JS实现首字母排序
date: 2019-11-22 16:56:10
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
    - Vue
tags:
  - 前端
  - JavaScript
  - Vue
  - 排序
---

<!-- <meta name="referrer" content="no-referrer" /> -->

一个按照姓名首字母排序的功能,支持数字,字母,符号,中文混合排序
<!-- more -->
## 规则
* 首字为中文则转换成小写
* 大小写均转换成小写比较,避免任意大写字母排在任意小写字母前
* 大写字母 == 小写字母 > 数字 > 符号 (这里数字和符号可以在最前面,因为业务需要被我拿到了最后面)

## 效果


```
// 排序前
this.friends = [
    {nick_name: 'cc'},
    {nick_name: 'Xin'},
    {nick_name: 'xiao李zi'},
    {nick_name: '__大熊'},
    {nick_name: 'Tian'},
    {nick_name: '3day'},
    {nick_name: '第一帅'},
    {nick_name: '大猪蹄子'},
];
// 排序后
this.friends = [
    {nick_name: 'cc'},
    {nick_name: '第一帅'},
    {nick_name: '大猪蹄子'},
    {nick_name: 'Tian'},
    {nick_name: 'Xin'},
    {nick_name: 'xiao李zi'},
    {nick_name: '3day'},
    {nick_name: '__大熊'},
];
```


## Library
* [hotoo/pinyin](https://hotoo.github.io/pinyin/)

## 实现
1. 安装__hotoo/pinyin__
```
npm install pinyin -S
```

2. 引入
```
import pinyin from 'pinyin';
```

3. 逻辑
{% codeblock lang:javascript %}
/**
 * 根据姓名或昵称排序,
 * 维护两个数组,分别存入字母开头和数字符号开头的项
 * 这里如果想把数字符号开头的名字排在最前面,就只用一个w_users就可以了,排序的时候会默认在最前面
 * @param  {Array} list 需要排序的数组对象,格式类似上面效果中的数据
 * @return {Array} sort_list 排序后的数组对象
 */
function name_sort(list) {
    let w_users = []; //字母开头
    let n_users = []; //数字 or 符号

    list.forEach(item => {
        let first_word = pinyin(item.nick_name, {
            style: pinyin.STYLE_FIRST_LETTER
        });
        
        first_word = first_word[0][0];
        first_word = first_word.substr(0, 1);
        
        item.nick_name_sort = first_word + item.nick_name;//增加一个临时排序属性, 不然排序后还要处理原属性
        let regx = /^[A-Za-z]*$/; //正则匹配出字母开头的
        let flag = regx.test(first_word);
        flag && w_users.push(item) || n_users.push(item); //根据类型决定存入哪个数组
    })

    //利用新增的nick_name_sort排序
    w_users.sort((a,b) => a.nick_name_sort.substr(0, 1).toLowerCase().charCodeAt(0) - b.nick_name_sort.substr(0, 1).toLowerCase().charCodeAt(0));
    n_users.sort((a,b) => a.nick_name_sort.substr(0, 1).toLowerCase().charCodeAt(0) - b.nick_name_sort.substr(0, 1).toLowerCase().charCodeAt(0));
    let sort_list = w_users.concat(n_users);
    return sort_list;
}

{% endcodeblock %}