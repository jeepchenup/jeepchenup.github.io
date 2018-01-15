---
layout: post
title:  "JavaScript 函数"
subtitle: ""
date:  2018-01-15
author: "ABei"
catalog: true
tags: 
    - JavaScript
    - Summary
---

#### 函数定义

> 任何合法的JavaScript标识符都可以作为一个函数的名称。函数名称标识符通常有三种：驼峰法、下划线分隔和下划线前缀(通常用作内部函数或私有函数)。

定义一个函数其实有三个部分来组成：函数名称标识符[可选]、一对括号和一对花括号。为什么说函数名称标识符是可以忽略呢？当函数用作表达式的时候，函数名称标识符就可省略了。

#### 函数调用

> 如果函数挂载在一个对象上，作为对象的一个属性，就称它为对象的方法。当通过这个对象来调用函数时，该对象就是此次调用的上下文(context)，也就是该函数的this的值。换句话说，任何函数只要作为方法调用实际上都会传入一个隐式的实参——就是这个对象(this)。
> 用于初始化一个新创建的对象的函数被称作构造函数。

函数调用有四种方式：
- 作为函数
- 作为方法
- 作为构造函数
- 通过他们的call()和apply()方法间接调用

在通读文本之前，我区分不出函数和方法两者具体有什么不一样？我认为函数就是方法，方法就是函数。在Java中或许是这样，但是JavaScript中还是有区别的。

```javascript
//define a function
function difference() {
    console.log("call function: difference");
}
//作为函数
difference();
//作为方法
var a = {};
a.fangfa = difference;
a.fangfa();
```