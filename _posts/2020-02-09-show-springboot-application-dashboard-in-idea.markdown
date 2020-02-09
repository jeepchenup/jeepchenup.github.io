---
layout: post
title: "Intellij Idea Show SpringBoot Application Dashboard"
author: ABei
categories: IDE
tags: IDEA
---
* content
{:toc}

最近在用IDEA开发，微服务多了之后想要通过IDEA进行批量开启服务。



记忆中是记得有那么一个操作的，但是按照网上一些教程操作之后发现还是无法实现。直到今天早上才发现怎么操作。

首先，来看下我原来的运行配置是怎么样的？

原先每个服务我都是通过新建Application类别的运行配置，进行运行微服务的。

![](/images/2020-02-09_101712.png)

然后，今早我将原来的运行配置删除了，直接执行通过SpringBoot启动类启动微服务。

![](/images/2020-02-09_103004.png)

发现这样启动多个微服务之后，就会出现Spring Boot Dashboard界面了。看了一下运行配置类别自动检测到SpringBoot上面。终于发现了区别的地方了。

![](/images/2020-02-09_103703.png)

按照上面配置之后就能看到Spring Boot Dashboard界面了。

![](/images/2020-02-09_103948.png)