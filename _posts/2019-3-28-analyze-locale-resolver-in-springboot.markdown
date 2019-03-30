---
layout: post
title: "浅析 LocaleResolver 在 SpringBoot 的执行流程"
author: ABei
categories: Framework
tags: SpringBoot Logback
---
* content
{:toc}

最近打算用 SpringBoot 写一个简单 CRUD 功能。在实现国际化的过程中，我觉得 SpringBoot 自动配置的功能不满足我的需求。所以，我就着手自己写了一个 LocaleResolver。



## 问题

在实现完毕之后，我发现重载方法 `resolveLocale(HttpServletRequest request)` 中的日志打印了 2 次。

![](http://cdn.51leif.com/2019-3-28-issue-log.png)

```java
public class CustomLocaleResolver implements LocaleResolver {
    ... ...

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        Optional<String> localeStr = Optional.ofNullable(request.getParameter(language));
        Locale locale = null;
        try {
            String[] localeInfo = localeStr.get().split("_");
            locale = new Locale(localeInfo[0], localeInfo[1]);
            // 问题所在处
            logger.info("Get Custom locale language :{}", localeStr.get());
        } catch (NoSuchElementException e) {
            logger.warn("There is no language parameter in request.", e);
            locale = Locale.getDefault();
        }

        return locale;
    }

    ... ...
}
```

## 排查思路

1.  是不是 `logback.xml` 配置文件有问题？

    ```xml
    <logger name="info.chen.smallcrud" level="info" additivity="false">
        <appender-ref ref="stdout" />
    </logger>
    ```

    发现自己指定的 logger 是正确的，`additivity="false"`。如果将 `additivity` 设置成 true，那么打印出来就是 4 次了。

    那么，看来不是日志出问题了。

2.  打断点排查，看看是哪些地方调用了 `resolveLocale` 方法。

    将断点打在 `CustomLocaleResolver.resolveLocale(HttpServletRequest request)`。

    ![](http://cdn.51leif.com/2019-3-28-debug-1.png)

    发现确实有 2 处地方调用了 `resolveLocale` 方法。不过这 2 处地方都是在 `DispatcherServlet.render()` 中被调用了。

    ```java
    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
		// 第一处调用
		Locale locale =
				(this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
		... ...
		try {
			if (mv.getStatus() != null) {
				response.setStatus(mv.getStatus().value());
			}
            // 第二处调用
			view.render(mv.getModelInternal(), request, response);
		}
		
        ... ...
	}
    ```

3.  是不是自己重写的有误？看看默认情况下，会不会有这种情况发生。

    SpringBoot 中默认的 LocleResolver 的实现类是 `AcceptHeaderLocaleResolver`。 经过打断点排查，发现也是和上面自定义的 LocaleResolver 一样会进 2 次。

## 总结

看了 DispathcerServlet 源码之后，发现打印 2 次相同的日志和 SpringBoot 并没有太大的关系。反而是和 DispatcherServlet 中的 `render` 方法有很大的关系。

原来这个情况并不是实现上的问题，也不是 render 设计上的问题。为什么这么说呢？因为，作者的意图也很明显，他是想将 Locale 对象存放在两处，一个是 `Response` 中，另一个是 `View` 视图中。所以才有了 2 次调用的情况。