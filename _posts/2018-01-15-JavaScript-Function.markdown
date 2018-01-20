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

> 如果函数挂载在一个对象上，作为对象的一个属性，就称它为对象的方法。当通过这个对象来调用函数时，该对象就是此次调用的上下文(context)，也就是该函数的this的值。换句话说，任何函数只要作为方法调用实际上都会传入一个隐式的实参——就是这个对象(this)。<br>
> 用于初始化一个新创建的对象的函数被称作构造函数。

函数调用有四种方式：
- 作为函数
- 作为方法
- 作为构造函数
- 通过他们的call()和apply()方法间接调用

在通读文本之前，我区分不出函数和方法两者具体有什么不一样？我认为函数就是方法，方法就是函数。在Java中或许是这样，但是JavaScript中还是有区别的。

```javascript
//debug in chrome
//define a function
function difference() {
    console.log("call function: difference");
}
//作为函数
difference();
//作为方法
var a = {};
a.fangfa = difference; //函数作为值
a.fangfa();
```

#### 函数的实参和形参

> JavaScript中的函数定义并未指定函数形参的类型，函数调用也未对传入的实参值做任何类型检查。<br>
> 同时，JavaScript函数也不会对参数的个数进行检查。

- 可选形参
    - 全称为“形式参数”，实际是不存在的变量。
    - 当传入的参数小于函数声明时指定的个数时，JavaScript函数会将剩余的形参都设置为undefined值。
- 实参
    - 全称为“实际参数”是在调用是传递给函数的参数，他们必须都有确定的值。
    - 在函数体内，标识符*arguments*是指向*实参对象*的引用。注意：arguments并不是真正的数组，它是一个实参对象。每个实参对象都包含*以数字为索引的一组元素和length属性*。
    - 在ES5中已经将*arguments*这个特殊移除。在严格模式中，arguments是一个保留字，而在非严格模式中，它仅仅是一个标识符。
    - callee和caller属性。这两个属性也是存于*实参对象*。两者在严格的ES5下，都将产生一个**类型错误**。
        - callee是ECMS标准规范制定的，返回当前正在执行的函数。
        - caller是非标准的(大部分的浏览器都实现了)，返回当前调用正在执行的函数的函数。

```javascript
//debug in chrome
function actualParameter(x) {
    console.log(x);        //输出实参的初始值
    arguments[0] = null;   //对实参对象的修改也会对x进行修改
    console.log(x);        //输出'null'
    console.log(arguments.callee); //返回当前函数内容
}
```

#### 作为namespace的函数

> 不要污染全局命名空间。

命名空间的函数的目的就是将同一块JavaScript代码，复用于不同的JavaScript程序中。

```javascript
(function(){/* 业务逻辑 */}()); //这样的匿名函数不会污染全局
```

#### 闭包(closure)

> 闭包，函数对象可以通过作用域链相互关联起来，函数体内的变量都可以保存在函数作用域内。

从上面的概念可知，闭包是和函数对象的作用域相关的，这也是闭包的核心。光看概念非常抽象，我所理解的闭包就是**能够读取其他函数内部变量的函数**。

要理解闭包之前，必须得理解变量作用域。变量的作用域无非就2种：全局变量和局部变量。

JavaScript语言的特殊之处，就在于函数内部可以直接读取全局变量。
```javascript
var scope = "global scope";

function checkscope() {
    console.log(scope);
}

checkscope(); // global scope
```

反过来，在函数外部自然无法读取函数内部的局部变量。

```javascript
function checkscope() {
    var scope = "local";
}

console.log(scope); //undefined
```

但是外部函数真的不能访问函数内部的局部变量吗？其实是可以的，这也就是闭包的作用。

```javascript
function checkscope() {
    var scope = "local";

    function checkLocal() {
        console.log(scope);
    }

    return checkLocal;  //将checkLocal作为返回值
}

var readLocal = checkscope();
readLocal(); // local
```

本来内部的局部变量对于外部来说是隐藏的。上面的代码，实现了外部读取函数内部的局部变量。checkscope这个函数就是**闭包**。在本质上，闭包就是将函数内部和函数外部连接起来的桥梁。

闭包最大的作用有两个：
- 就是前面讲到的，将函数内部和函数外部连接起来。
- 让函数内部的变量始终保持在内存中。

如何来理解闭包的第二个作用？

```javascript
function checkNum() {
    var num = 0;

    function checkLocal() {
        console.log(num++);
    }

    return checkLocal;  
}

var readNum = checkNum();

readNum();
readNum();
```