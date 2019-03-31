---
layout: post
title: "解析 Spring Controller 映射方法参数的映射过程"
author: ABei
categories : Framework
tags: SpringMVC
---
* content
{:toc}

> Spring Version: 5.1.5.RELEASE

使用 SpringMVC 也比较久了，心中一直有个困惑，就是 Controller 中的请求映射方法参数到底是如何被绑定上去的？




## 猜测

我想，Spring 容器应该事先设定了一些可以映射的参数供 Controller 中的请求映射方法使用，接着在反射过程中，将这些值传递进去。为了印证我的猜测，请看下面的源码分析。

## 分析

下面是一段常见 Controller 的写法。在 `LoginController` 中，定义了一个映射方法 `login`。其中，我写了 5 个映射参数。让我们来看看 Spring 容器是如何帮我们绑定参数的呢？是不是任意的参数 Spring 容器都能一一将其对应起来呢？

```java
@Controller
public class LoginController {
    
    ... ...

    @PostMapping("/login")
    public String login(@RequestParam("username") String username, 
                        @RequestParam("password") String password, 
                        Map<String, Object> map, 
                        HttpSession session, 
                        HttpServletRequest request, 
                        RedirectAttributes redirectAttributes) {
        ... ...
    }

}
```

为了本文清晰明了一点，我将通过下面几个重要的类入手来分析 Spring 容器是如何绑定映射方法的参数的。

1.  org.springframework.web.method.**HandleMethod**
1.  org.springframework.web.method.support.**InvocableHandlerMethod**
1.  org.springframework.web.method.support.**HandlerMethodArgumentResolverComposite**

### 1. HandlerMethod

Spring 容器在初始化 Controller 的时候，会调用 `HandlerMethod.initMethodParameters()` 方法。通过这个方法，为每个 Controller 中的映射方法的参数 **初始化对应的** `MethodParameter` 对象。

```java
private MethodParameter[] initMethodParameters() {
    int count = this.bridgedMethod.getParameterCount();
    MethodParameter[] result = new MethodParameter[count];
    for (int i = 0; i < count; i++) {
        // 初始化 MethodParameter 对象
        HandlerMethodParameter parameter = new HandlerMethodParameter(i);
        // 解析对应的方法参数类型，也就是设置 MethodParameter 的 parameterType 属性
        GenericTypeResolver.resolveParameterType(parameter, this.beanType);
        result[i] = parameter;
    }
    return result;
}
```

### 2. InvocableHandlerMethod

我们先来看看，在我们请求 URI `/login` 时的方法调用栈图，如下所示：

![](http://cdn.51leif.com/2019-3-31-data-resolver-call-stack.svg)

从这个方法调用栈，我们可以看到，在方法执行到 `InvocableHandlerMethod.getMethodArgumentValues` 方法时，开始了真正的参数绑定过程。

```java
/**
 * @parameter providedArgs, 在我打断点的时候为 null，所以不用管
 */
protected Object[] getMethodArgumentValues(NativeWebRequest request,
                                           @Nullable ModelAndViewContainer mavContainer,
                                           Object... providedArgs) throws Exception {

    if (ObjectUtils.isEmpty(getMethodParameters())) {
        return EMPTY_ARGS;
    }

    // 获取对应映射方法的 MethodParameter 数组
    MethodParameter[] parameters = getMethodParameters();
    Object[] args = new Object[parameters.length];

    // 开始遍历每个 MethodParameter 对象，
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
        
        // 检查当前参数类型是否已经提供了参数值
        args[i] = findProvidedArgument(parameter, providedArgs);
        if (args[i] != null) {
            // 如果已经提供参数值，那么直接返回
            continue;
        }

        /**
         * 注意，这里很关键
         * 通过方法名 supportsParameter，大概可以推断出这个方法是用于
         * 检查当前 MethodParameter 对象的参数类型(parameterType 属性)
         * 是不是能被 InvocableHandlerMethod.resolvers 集合中的解析器解析。
         * 如果找到对应的 ArgumentResolver，
         * 将会将这个解析器实例和 MethodParameter 对象放入 argumentResolverCache 中
         *
         * 至于，InvocableHandlerMethod.resolvers 有哪些解析器，什么时候被初始化的？后面会讲到
         */
        if (!this.resolvers.supportsParameter(parameter)) {
            throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
        }
        try {
            // 进行数据绑定
            args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
        } catch (Exception ex) {
            // Leave stack trace for later, exception may actually be resolved and handled..
            if (logger.isDebugEnabled()) {
                String error = ex.getMessage();
                if (error != null && !error.contains(parameter.getExecutable().toGenericString())) {
                    logger.debug(formatArgumentError(parameter, error));
                }
            }
            throw ex;
        }
    }
    return args;
}
```

### 3. HandlerMethodArgumentResolverComposite

在 `HandlerMethodArgumentResolverComposite.resolveArgument` 进行数据绑定工作。

```java
public Object resolveArgument(MethodParameter parameter,
                              @Nullable ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest,
                              @Nullable WebDataBinderFactory binderFactory) throws Exception {

    // 根据 MethodParameter 找到对应的方法参数解析器
    HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
    if (resolver == null) {
        throw new IllegalArgumentException(
                "Unsupported parameter type [" + parameter.getParameterType().getName() + "]." +
                        " supportsParameter should be called first.");
    }

    /**
     * 进行数据绑定，这里就不再深入，
     * 大体上，整个流程已经分析完，
     * 有兴趣可以自己再深入。
     */
    return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

### 4. InvocableHandlerMethod.resolvers 初始化时机

上面，已经分析完整个 Controller 映射方法参数的绑定过程。下面来，看看 Spring 容器为我们提供了哪些默认的 `ArgumentResolver`。

在 Spring 容器初始化 `org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter` 的时候，会间接的初始化了 `InvocableHandlerMethod`。

具体方法是在 `RequestMappingHandlerAdapter.afterPropertiesSet()` 中被初始化。

```java
public void afterPropertiesSet() {
    // Do this first, it may add ResponseBody advice beans
    initControllerAdviceCache();

    if (this.argumentResolvers == null) {
        // 获取默认的 ArgumentResolvers
        List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
        this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.initBinderArgumentResolvers == null) {
        List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
        this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.returnValueHandlers == null) {
        List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
        this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
    }
}
```

`RequestMappingHandlerAdapter.getDefaultArgumentResolvers()` 方法

```java
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
    List<HandlerMethodArgumentResolver> resolvers = new ArrayList<>();

    // Annotation-based argument resolution
    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
    resolvers.add(new RequestParamMapMethodArgumentResolver());
    resolvers.add(new PathVariableMethodArgumentResolver());
    resolvers.add(new PathVariableMapMethodArgumentResolver());
    resolvers.add(new MatrixVariableMethodArgumentResolver());
    resolvers.add(new MatrixVariableMapMethodArgumentResolver());
    resolvers.add(new ServletModelAttributeMethodProcessor(false));
    resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new RequestHeaderMapMethodArgumentResolver());
    resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new SessionAttributeMethodArgumentResolver());
    resolvers.add(new RequestAttributeMethodArgumentResolver());

    // Type-based argument resolution
    resolvers.add(new ServletRequestMethodArgumentResolver());
    resolvers.add(new ServletResponseMethodArgumentResolver());
    resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RedirectAttributesMethodArgumentResolver());
    resolvers.add(new ModelMethodProcessor());
    resolvers.add(new MapMethodProcessor());
    resolvers.add(new ErrorsMethodArgumentResolver());
    resolvers.add(new SessionStatusMethodArgumentResolver());
    resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

    // Custom arguments
    if (getCustomArgumentResolvers() != null) {
        resolvers.addAll(getCustomArgumentResolvers());
    }

    // Catch-all
    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
    resolvers.add(new ServletModelAttributeMethodProcessor(true));

    return resolvers;
}
```

下次，再想到给映射方法传参的时候就不会迷惑了，去这里找找看就知道有没有对应的参数解析器即可。