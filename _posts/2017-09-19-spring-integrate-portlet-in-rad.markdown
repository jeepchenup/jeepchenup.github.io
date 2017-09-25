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

> Spring integrate portlet in RAD9.0<br/>
> [Referenced document](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/portlet.html)

- 预计阅读时间: 5分钟

## 内容简介

* 创建
    - [portlet](#portlet)
    - [portletname-portlet.xml](#portletXML)
    - [applicationContext.xml](#applicationContextXML)
* 配置
    - [web.xml](#webXML)
    - [portlet.xml](#portletXML)

### Portlet

*Portlet* 可以手动创建 *Portlet* 类也可以用 *RAD* 模板创建，这里我选择模板创建。模板创建有个好处就是可以不需要手动创建 *nl* 包。
代码如下：
```java
@Controller
@RequestMapping("VIEW")
public Class CusNameDisplayPortalViewController {

    @RequestMapping
    public String handleRenderRequestInternal(RenderRequest request) throws Exception {
        String str = "Hello Portlet";
        request.setAttribute("hello", str);
        return "display";
    }
}
```

模板生成nl包需要移动到`resources`目录下面
![nl包路径](/img/in-post/spring-integrate-portlet/path-struc.png)
`CusNameDisplayPortalViewController.properties`内容如下

```xml
javax.portlet.title=CusNameDisplayPortalViewController
javax.portlet.short-title=CusNameDisplayPortalViewController
javax.portlet.keywords=CusNameDisplayPortalViewController
```