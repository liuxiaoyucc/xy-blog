---
title: ES6中的import和export使用方法
date: 2020-04-06 13:35:09
categories:
  - - 大熊BIGBEAR
    - 前端
    - JavaScript
tags:
  - Js
  - 随手记
---

<meta name="referrer" content="no-referrer" />

1. 
```
export const obj = {};
export const obj2 = {};

import * as JsName from './jsName.js'; // obj和obj2均在JsName对象下

```

2. 
```
export const obj = {};
export const obj2 = {};

import { obj } from './schedule.js';
import { obj2 } from './schedule.js';

```
