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

### 内容简介

* 创建
    - [portlet](#portlet)
    - [portlet配置文件](#properties)
    - [portletname-portlet.xml](#portletName)
    - [applicationContext.xml](#appContext)
* 配置
    - [web.xml](#web)
    - [portlet.xml](#portlet)
* 源码
    - [点击下载](https://github.com/jeepchenup/jeepchenup.github.io/blob/master/_sources/2017.09.19/integrateSpring.7z)

### 目录结构

先看一下整个项目的结构，注意在创建 *Protlet* 项目的同时也创建相应的 *EAR* 项目：
![SpringIntegratePortletProject](/img/in-post/spring-integrate-portlet/project-struc.png)

#### Portlet

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

#### Properties

`CusNameDisplayPortalViewController.properties`&rarr;

```xml
javax.portlet.title=CusNameDisplayPortalViewController
javax.portlet.short-title=CusNameDisplayPortalViewController
javax.portlet.keywords=CusNameDisplayPortalViewController
```

#### AppContext

文件放置位置：`WEB-INF`<br>
这里我在该目录下面创建了一个`context`的子目录（**注意 *applicationContext.xml* 文件路径**）

```xml
<context:annotation-config />

<bean id="messageSource" 
    class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basenames">
        <list>
            <!-- Add appropriate resource properties path -->
            <value>com.controllers.nl.FirstSpringPortletResource</value>
        </list>
    </property>
</bean>
<!-- Default View Resolver -->
<bean id="viewResolver" 
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass">
        <value>org.springframework.web.servlet.view.JstlView</value>
    </property>
    <property name="prefix">
        <value>/WEB-INF/jsp/</value>
    </property>
    <property name="suffix">
        <value>.jsp</value>
    </property>
</bean>
```

#### Web

对于`web.xml`主要是对配置上面创建的`application.xml`进行路径设置，以及配置 *Spring* 的监听器。

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/context/applicationContext.xml</param-value>
</context-param>
```