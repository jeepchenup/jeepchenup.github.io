---
layout: post
title:  "JavaScript 对象"
subtitle: ""
date:  2017-12-14
author: "ABei"
catalog: true
tags: 
    - JavaScript
    - Summary
---

> 由于工作需要，最近在读老司机推荐的《JavaScript权威指南》。

### 我对JavaScript对象的理解

在对象这章的开头，书中就介绍到对象常见的操作有：create、delete、set、query、test、enumerate。这些操作都是对其**属性**的操作。属性是对象的*核心*，它能准确的表达对象，对象其实也就是属性的集合。在我看来，对象也是由各种JSON组成的。

### 对象的属性

属性由名字、值和一组特性组成（property = name + value + attribute）。name可以是包含**空字符串**在内的任意字符串，value是任意的。

> 一个对象里面不能有两个相同属性名。

说是说不能有两个同名的属性，但在JavaScript中实现会是什么情况呢？
```javascript
//in chrome
var o = {"":2, "": 3};
o// Object {"": 3}
```

在对JavaScript对象赋值属性的时候，如果出现同名属性，后面的属性会覆盖前面的属性。

### 对象属性的2中表现形式

1. 数据属性（data property）。上面提及的都是数据形式。数据属性只有一个简单的值。
2. 存取器属性（accessor property）。属性的表现形式用两个方法代替: getter和setter。

```javascript
var o = {
    //数据属性
    data_prop: "value",

    // 存取器属性都是成对定义的函数
    get accessor_prop() { return this.data_prop;},
    set accessor_prop(value) { this.data_prop = value;}
}

//读取data_prop
//方法1
o.data_prop;
//方法2
o.accessor_prop;
//设置data_prop参数
//方法1
o.data_prop = "value2";
//方法2
o.accessor_prop = "value2";
```

存取属性是成对出现，但不代表一定得成对出现。

```javascript
var o = {
    //数据属性
    data_prop: "value",

    // 存取器属性都是成对定义的函数
    get accessor_prop() { return this.data_prop;}
}

o.accessor_prop = "value2"; //o.data_prop = "value"
```

从上面可以看出，o对象只有get方法，通过`o.accessor_prop = "value2";`虽然不会报错，但是并不会修改`o.data_prop`的值。

### 删除属性

delete运算符可以删除对象的属性。其实就是断开属性与宿主的联系，并不会对属性的属性进行操作。

```javascript
var c = {child : "child"};
var p = {parent: "parent", child:c};
function f() {};
delete p.child; // true
c // Object {child: "child"}
delete p; // false
delete f; // false
```

delete可以删除对象的自有属性，但是不能删除全局变量以及全局函数。

### 对象的特性

一个对象除了包含属性以外，还拥有三个相关的对象特性：prototype（原型）、class、扩展标记（extensible flag）。
1. 原型。在我看来，就相当于Java里面的Object类，是所有类的终极父类（除了接口，详细请看[链接](https://docs.oracle.com/javase/specs/jls/se7/html/jls-9.html#jls-9.6.3.4)）。每一个JavaScript的对象，都会从原型那里继承属性。
    - 通过**直接量**创建的对象，以Object.prototype作为原型。
    - 通过**new**创建的对象，以该对象的构造函数的prototype属性作为原型。
    - 通过**Object.create()**创建的对象，使用第一个参数（可以是null）作为原型。

2. class。是以字符串作为展现形式，用来表示对象的类型信息。
```javascript
function classOf(o) {
    return Object.prototype.toString.call(o).slice(8,-1);
}
```

3. 可扩展性。正如字面上的意思，就是表示当前对象是否可以添加新的属性。

### 对象序列化

对象序列化，是指将对象的状态转化为字符串，也可以将字符串还原为对象。可以通过下面2中方法对对象进行序列化和还原。

1. 序列化。JSON.stringify()，该方法内部调用JSON.toJSON();

2. 还原。JSON.parse();
