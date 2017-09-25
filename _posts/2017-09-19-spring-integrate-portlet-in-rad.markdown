---
layout: post
title:  "SpringMVC Integrate Portlet"
subtitle: "In RAD9.0"
date:  2017-09-19
author: "Steven"
catalog: true
tags: 
    - Portlet
    - SpringMVC
---

> Spring3.0 integrate portlet in RAD9.0<br/>
> [Referenced document](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/portlet.html)

- 预计阅读时间: 15分钟

## 内容简介

* 创建
    - [portlet](#portlet)
    - [portlet配置文件](#properties)
    - [portletname-portlet.xml](#portletXML)
    - [applicationContext.xml](#applicationContextXML)
* 配置
    - [web.xml](#webXML)
    - [portlet.xml](#portletXML)
* 源码
    - [点击我](https://jeepchenup.github.com/jeepchenup.github.io)

## 目录结构

先看一下整个项目的结构，注意在创建 *Protlet* 项目的同时也创建相应的 *EAR* 项目：
![SpringIntegratePortletProject](/img/in-post/spring-integrate-portlet/project-struc.png)

### Portlet

*Portlet* 可以手动创建 *Portlet* 类也可以用 *RAD* 模板创建，这里我选择模板创建。模板创建有个好处就是可以不需要手动创建 *nl* 包。
代码如下：
```java
@Controller
@RequestMapping("VIEW")
public class FirstSpringPortlet {
	
	@RequestMapping
	public String showIndex(RenderRequest request) {
		request.setAttribute("name", "Steven");
		return "index";
	}
}
```

### Properties

`CusNameDisplayPortalViewController.properties`<br>
&darr;&darr;

```xml
javax.portlet.title=CusNameDisplayPortalViewController
javax.portlet.short-title=CusNameDisplayPortalViewController
javax.portlet.keywords=CusNameDisplayPortalViewController
```

### ApplicationContextXML

文件放置位置：`WEB-INF`<br>
这里我在该目录下面创建了一个`context`的子目录。注意 *ApplicationContext.xml* 文件路径。
