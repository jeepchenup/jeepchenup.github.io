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

- 预计阅读时间: 7分钟

### 内容简介

* 创建
    - [portlet](#portlet)
    - [portlet配置文件](#properties)
    - [portletname-portlet.xml](#portlet-name-xml "portletname-portlet.xml")
    - [applicationContext.xml](#application-context-xml "applicationContext.xml")
* 配置
    - [web.xml](#web-xml "web.xml")
    - [portlet.xml](#portlet-xml "protlet.xml")
* [总结](#总结 "summary")
* 源码
    - [点击下载](https://github.com/jeepchenup/jeepchenup.github.io/blob/master/_sources/2017.09.19/integrateSpring.7z "快点我呀")

### 目录结构

先看一下整个项目的结构，注意在创建 *Protlet* 项目的同时也创建相应的 *EAR* 项目：
![SpringIntegratePortletProject](/img/in-post/spring-integrate-portlet/project-struc.png)

#### Portlet

*Portlet* 可以手动创建 *Portlet* 类也可以用 *RAD* 模板创建，这里我选择模板创建。模板创建有个好处就是可以不需要手动创建 *nl* 包并且自动生成 *portlet.xml* 文件。
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

`FirstSpringPortlet.properties`&rarr;

```xml
javax.portlet.title=FirstSpringPortlet
javax.portlet.short-title=FirstSpringPortlet
javax.portlet.keywords=FirstSpringPortlet
```

#### Application Context XML

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

#### Web XML

对于`web.xml`主要是对配置上面创建的`application.xml`进行路径设置，以及配置 *Spring* 的监听器。

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/context/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

#### Portlet XML

*Portlet XML* 在创建 *Portlet* 类的时候就自动生成了，但是与 *Spring MVC* 的整合，需要将`<portlet-class>`引用的类改成 `org.springframework.web.portlet.DispatcherPortlet`

```xml
<portlet>
    <portlet-name>FirstSpringPortlet</portlet-name>
    <display-name>FirstSpringPortlet</display-name>
    <display-name xml:lang="en">FirstSpringPortlet</display-name>
    <portlet-class>org.springframework.web.portlet.DispatcherPortlet</portlet-class>
    <init-param>
        <name>wps.markup</name>
        <value>html</value>
    </init-param>
    <expiration-cache>0</expiration-cache>
    <supports>
        <mime-type>text/html</mime-type>
        <portlet-mode>view</portlet-mode>
    </supports>
    <supported-locale>en</supported-locale>
    <resource-bundle>com.controllers.nl.FirstSpringPortletResource</resource-bundle>
    <portlet-info>
        <title>FirstSpringPortlet</title>
        <short-title>FirstSpringPortlet</short-title>
        <keywords>FirstSpringPortlet</keywords>
    </portlet-info>
</portlet>
```

#### Portlet Name XML

*Portlet MVC* 是一个请求驱动的web *MVC* 框架，它围绕着一个 *Portlet* 进行设计。*Portlet* 将请求发送到 *Controller*，并提供其他功能，以促进 *Portlet* 的应用程序开发。然而，*Spring* 的 *DispatcherPortlet* 除了实现上面讲到的功能之外，它还与 *Spring Application Context* 完全集成，允许你使用 *Spring* 的其他特性。

在 *Portlet MVC* 框架中，每个 *DispatcherPortlet* 都有自己的 *WebApplicationContext*，它继承了根 *WebApplicationContext* 中已经定义的所有bean。这些继承的 *beans* 可以在 *Portlet* 特定的范围内被重写，对于`new scope-specific` *beans*, 它们可以被定义到一个本地的 *Portlet* 实例。

在初始化 *DispatcherPortlet* 时，*Spring* 框架将在你的 *web application* 的`WEB-INF`目录下寻找一个命名为`[portlet-name]-portlet.xml`文件。然后根据这个xml文件，创建相应的 *bean* 实例。在 *Spring* 的官方文档里面写到，
> overriding the definitions of any beans defined with the same name in the global scope.

个人理解为，如果`WEB-INF`下面存在`[portlet-name]-portlet.xml`，*Spring MVC* 就不会去查找`root applicationContext`来创建bean实例。所以可以在全局范围内，重写任意定义过相同名字（与父类名称相同）的 *beans*。

创建相应的 *Portlet XML* 文件。这里的 *Portlet* 实则是 *Spring* 中的 *Controller*，`FirstSpringPortlet.class`并没有继承`javax.portlet.GenericPortlet`这个类。显然在部署阶段，项目是找不到`portlet.xml`中所指定portlet。*Spring* 给我们的解决方案就是，创建一个 *xml* 文件，文件命名规则有规定(**[portlet-name]-portlet.xml**)。*portletname* 就是`portlet.xml`中`<portlet-name>`中所设置的名称。`FirstSpringPortlet-portlet.xml`的关键配置如下：

```xml
<context:annotation-config/>

<!-- Controllers -->
<context:component-scan base-package="com.controllers" />

<!-- Handler Mappings -->
<bean class="org.springframework.web.portlet.mvc.annotation.DefaultAnnotationHandlerMapping"/>
```

#### 总结
