---
title: 测试代码块
date: 2019-11-12 11:06:58
categories:
    - 测试
tags:
    - 测试
---

## 普通代码块
{% codeblock %}
alert('Hello World!');
{% endcodeblock %}


## 指定语言

{% codeblock lang:objc %}
[rectangle setX: 10 y: 10 width: 20 height: 20];
{% endcodeblock %}

## 附加说明
{% codeblock Array.map %}
array.map(callback[, thisArg])
{% endcodeblock %}


```
function softbankToUnified(text) {
  return text.replace(
    createRegexp(EMOJI_SOFTBANK_MAP),
    (_, m) => EMOJI_SOFTBANK_MAP[m],
  );
}
```
