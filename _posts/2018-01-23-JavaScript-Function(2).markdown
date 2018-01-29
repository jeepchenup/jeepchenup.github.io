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

#### call()方法和apply()方法

> call()和apply()都是为了动态改变this而出现的，当一个object没有某个方法，但是其他的有，我们可以借助call或者apply用其他对象的方法来操作。

call的原理：
```javascript
var o = {};

function f() {
    console.log("call f function");
}

function sum(x,y,z) {
    if(arguments.length != 3)
        throw Error("arguments should be 3");
    console.log(x+y+z);
}

f.call(o); // 输出'call f function'
o // {};

sum.call(o,1,2,3) //输出6
sum.call(o,1)     //Uncaught Error: atguments should be 3

sum.apply(o,[1,2,3]);//输出6
sum.apply(o,[1]);    //Uncaught Error: atguments should be 3

/*下面的操作就相当于f.call(o)*/

o.temporaryAttribute = f;
o.temporaryAttribute();
delete o.temporaryAttribute;


window.number = 'one';
document.number = 'two';

var s1 = { number : 'three'};
function changeColor() {
    console.log(this.number);
}

changeColor().apply();          //one  
changeColor().apply(window);    //one
changeColor().apply(document);  //two
changeColor().apply(this);      //one  
changeColor().apply(s1);        //three
```

apply在功能上和call是类似的，但是传入实参的形式和call有所不同，第一个传入的都是当前要执行该方法的对象，后面是传入方法的实参，但是我们可以看到，apply的方式是数组(可以是真的数组，也可以是类数组)，这点与call方法是不一样的。

#### bind()方法

> bind方法的作用就是将函数绑定到某个对象。

```javascript
//待绑定的函数
function f(y) {
    return this.x + y;
}
var o = { x: 1};
var g = f.bind(o); // g(x) == o.f(x)
g(2); // => 3

o // {x: 1} 虽然f绑定到o上面但是，实际上o没有任何的变化

/*模拟bind实现过程*/
o.m = f; //
var g = o;
delete o.m;
```

bind方法看似就是将一个方法暂时赋值给了对象，方法结束就把函数解绑。实际上，在绑定的时候不仅仅将方法赋值给了对象，同时也将对象的this属性给绑定进去了。

#### Function()构造函数

> 除了通过关键字**function**来创建函数之外，还可以通过**Function()**构造函数来定义函数。

- Function()构造函数特点：
    - Function()构造函数允许JS脚本在运行的时候动态创建出来。
    - 每次调用Function()都会创建出新的函数对象。
    - 通过Function()构造函数创建出来的函数属于全局函数，所以不受创建脚本的环境影响。

```javascript
function keywordFun(x,y) {
    console.log(x,y);
    return x+y;
}

/*用Function来创建上面的keywordFun*/

var keywordFun = new Function("x","y", "console.log(x,y);return x+y;");

var scope = "global";
function constructFunction() {
    var scope = "local";
    return new Function("return scope;");
}

constructFunction()(); // => global;
```