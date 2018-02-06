---
layout: post
title:  "JavaScript 正则表达式"
subtitle: ""
date:  2018-01-29
author: "ABei"
catalog: true
tags: 
    - JavaScript
    - Summary
---

#### 定义正则表达式

在JavaScript中，定义正则表达式有两种方式：
- 第一种，就是**直接量定义**，变量包含在一对斜杠(`/`)之间的字符。
- 第二种，通过构造函数`RegExp()`来创建正则表达式变量。

```javascript
var pattern1 = /s$/;             //直接变量赋值
var pattern2 = new RegExp("s$"); //通过RegExp()创建
```