---
layout: post
title:  "工作总结(1)"
subtitle: ""
date:  2017-11-27
author: "Steven"
catalog: true
tags: 
    - JavaScript
    - Work
---

> 上个星期task完成的差不多的前提下，我让老司机帮我review code。结果还是有很多细节上的问题，感觉还是有很多地方需要改进。

#### 不足：

###### 关于读书的问题。

单个知识点是能理解，但是放在了一起，理解代码还是需要花一定的时间。并且在做task的时候，自己很容易会写那种冗余的代码。

比如：

```javascript
var sosStatus = false;
sosStatus = responseJSON.sosStatus == 'success' ? true : false;
```

上面的代码，使用三目运算符，显得多此一举了。如果不是老司机帮我review，估计我是很难发现这个问题。

###### 对于JavaScript数据类型与布尔值之间的转换不清晰。

```javascript
var isSosCapability = '${catViewMap.isSosCapability}';
if(isSosCapability) {
    //TODO true action 
} else {
    //TODO false action
}
```

由于后台不会传递空字符串，所以无论isSosCapability为何值都不会执行else里面的内容。写到这里，感觉自己平时看书的时候，是相当的不仔细。
下面的这6个值都是转换成false：

```javascript
undefined
null
0
-0
NaN
""//空字符串
```

###### 犯了太多低级而且相当致命的错误。

`sosStatus = responseJSON.sosStatus = 'success';`少写一个`'='`，还是发生在code review阶段，真想找个地方钻进去。
说到这里，我就写一下`'='`、`'=='`、`'==='`三者之间的区别。
- `'='`： 赋值
- `'=='`：如果两个操作数不是同一类型，那么相等运算符会尝试进行一些类型转换，然后进行比较。
    - `null == undefined //true`
    - 数字与字符串比较。先将字符串转换为数字，然后使用转后的值进行比较。
    - 和布尔值进行比较，先将布尔值转换成数字。`true`转换成1，`false`转换成0。
    - 一个值是对象，另外一个值为数字或字符串。先将对象转换成原始值，再进行比较。
    - 如果两个操作数的类型相同，遵循下面的`'==='`比较。
    - 其余不相同的操作数的类型情况下，结果均不相等。
- `'==='`：严格相等。
    - 两个值类型不相同，则它们不相等。
    - 两个值都是null或者都是undefined，则它们不相等。
    - 两个都是false或者都是true，则它们相等。
    - NaN与任何值(包括本身)相比较，结果都不相等。可以通过`x!==x`来判断是否为NaN，只有x为NaN的时候，这个表达式才为true。
    - 数字相比较，值相等，则它们相等。
    - 如果两个值为字符串，且所含的对应位上的16位数完全相等，则它们相等。
    - 如果两个引用指向同一个对象、数组或函数，则它们是相等的。否则不相等。