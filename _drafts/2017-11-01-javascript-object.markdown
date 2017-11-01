---
layout: post
title:  "Object"
subtitle: "JavaScript"
date:   2017-11-01
author: "Steven"
catalog: true
tags: 
    - JavaScript
---

> 对象是JavaScript的基本数据类型。对象是一种复合值：它将很多值（原始值或者其他对象）聚合在一起，可通过名字访问这些值。

### 内容简介

* [创建对象](#创建对象)

#### 创建对象

* 对象直接量
最简单的就是直接将对象直接量赋值给属性名?<br>
**注意：对于保留字的赋值，在ECMAScript3中需要添加双引号。**
```javascript
var empty = {};
var point = {x:0, y:0};
var book = {
    "main title": "JavaScript",           //属性名字里面有空格，必须用字符串表示
    'sub-title' : "The Definitive Guide", //属性名字里面有连字符，必须用字符串表示
    "for" : "all audiences",              //"for"是保留字，因此必须用引号
    author: {
        fisrtname : "Steven",
        lastname : "Chen"
    }
}
```

* 通过new创建对象
