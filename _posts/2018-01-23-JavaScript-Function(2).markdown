---
layout: post
title:  "JavaScript 函数(2)"
subtitle: ""
date:  2018-01-23
author: "ABei"
catalog: true
tags: 
    - JavaScript
    - Summary
---

> 函数是JavaScript中特殊的对象。因为函数也是对象，所以函数也可以拥有属性和方法，甚至可以通过**Function()**构造函数来创建新的函数对象。

#### length属性

函数的length属性也是*只读*的，它代表函数**形参**的数量，即函数声明的时候，所期望传入的实参的个数。
在函数体内，arguments.length表示传入函数的实际实参的个数。
下面用一段代码来判断函数的实参与期望的是否一致，不一致抛出异常。

```javascript
//debug in chrome
function check(args) {
    var actual = args.length;
    var expected = args.callee.length;
    if (actual != expected)
    throw Error("Expected " + expected + " args; got " + actual);
}

function sum(x,y,z) {
    check(arguments);
    return x+y+z;
}
sum(1+2);   // throw error info
sum(1+2+3); // return 6
```

#### call()方法

call的原理：
```javascript
var o = {};

function f() {
    console.log("call f function");
}

f.call(o); // 输出'call f function'
o // {};

/*下面的操作就相当于f.call(o)*/

o.temporaryAttribute = f;
o.temporaryAttribute();
delete o.temporaryAttribute;

```