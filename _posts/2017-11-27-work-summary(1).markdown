---
layout: post
title:  "工作总结(1)"
subtitle: ""
date:  2017-09-19
author: "Steven"
catalog: true
tags: 
    - JavaScript
    - Summary
---

> 上个星期task完成的差不多的前提下，我让老司机帮我review code。结果还是有很多细节上的问题，感觉还是有很多地方需要改进。

1. 首先是关于读书的问题，单个知识点是能理解，但是放在了一起，理解代码还是需要花一定的时间。并且在做task的时候，自己很容易会写那种冗余的代码。

比如：

```javascript
var sosStatus = false;
sosStatus = responseJSON.sosStatus == 'success' ? true : false;
```

上面的代码，使用三目运算符，显得多此一举了。如果不是老司机帮我review，估计我是很难发现这个问题。

2. 对于JavaScript数据类型与布尔值之间的转换不清晰。

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