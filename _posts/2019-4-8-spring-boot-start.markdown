---
layout: post
title: "分析 Spring Boot 启动过程"
author: ABei
categories : Framework
tags: SpringBoot
---
* content
{:toc}

> 重要的回调类：
>
> 1. ApplicationContextInitializer
> 2. SpringApplicationRunListener
> 3. ApplicationRunner
> 4. CommandLineRunner




## 1. 整体流程

1. 从标注 `@SpringBootApplication` 的类开始。
2. 调用 `SpringApplication.run(Class<?> primarySource,String... args)` 方法，细节如下：
   1. 创建 SpringApplication 对象，主要是初始化了 `initializers` 和 `listeners`。
   2. 调用 SpringApplication 对象的 `run()` 方法

## 2. 细节

### 2.1 创建 SpringApplication 对象

最终调用的构造函数：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    /**
     * 获取当前 web容器的类型
     * 1. Servlet 容器
     * 2. Reactive Web 容器
     * 3. None, 普通jar包
     */
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    /**
     * 将从 'META-INF/spring.factories' 文件中读取到的
     * ApplicationContextInitializer 的类实例化，
     * 并且保存到 initializers 变量中
     */
    setInitializers((Collection)
    getSpringFactoriesInstances(ApplicationContextInitializer.class));
    
    /**
     * 将从 'META-INF/spring.factories' 文件中读取到的
     * ApplicationListener 的类实例化，
     * 并且保存到 listeners 变量中
     */
    setListeners((Collection)getSpringFactoriesInstances(ApplicationListener.class));
    
    // 获取拥有 main 方法的类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

上面代码找中，有 2 个重要的方法：

1. `setInitializers`
2. `setListeners`

上面 2 个方法中都会调用 `getSpringFactoriesInstances` 方法：

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
    // 获取类加载器
    ClassLoader classLoader = getClassLoader();
    // 调用 `SpringFactoriesLoader.loadFactoryNames` 方法获取 type 类型的 classes 名字
    /**
     * 读取 classpath下的 'META-INF/spring.factories' 文件
     */
    Set<String> names = new LinkedHashSet<>(
        SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 初始化 names 中的类
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
                                                       classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

### 2.2 执行 SpringApplication 对象的 `run` 方法

```java
public ConfigurableApplicationContext run(String... args) {
    // 创建秒表
    StopWatch stopWatch = new StopWatch();
    // 开始计时
    stopWatch.start();
    // context 就是 IoC 容器
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    
    /**
     * 从 'META-INF/spring.factories' 中，
     * 获取并初始化 SpringApplicationRunListeners 全限定名
     * 所指向的类，最后初始化出 SpringApplicationRunListeners 对象
     */
    SpringApplicationRunListeners listeners = getRunListeners(args);
    
    // 遍历 listeners，回调 SpringApplicationRunlisteners 中的 listener.strarting() 方法
    listeners.starting();
    try {
        // 封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        
        // 配置环境变量
        /**
         * 1. 配置 IoC 容器所需的环境变量
         * 2. 回调 SpringApplicationRunlisteners 中的 listener.environmentPrepared
         * 3. 将环境变量绑定到 SpringApplication 中
         */
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                                                               applicationArguments);
        // 在环境变量中设置 IGNORE_BEANINFO_PROPERTY_NAME 的值
        configureIgnoreBeanInfo(environment);
        
        // 打印 Spring Boot 的横幅
        Banner printedBanner = printBanner(environment);
        
        // 根据 web application 类型, 创建(还不是初始化)相应的 IoC 容器
        context = createApplicationContext();
        
        // 获取异常分析报告类
        exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
        // 重要
        // 准备初始化 IoC 容器所需的上下文，详细分析请往下看
        prepareContext(context, environment, listeners, applicationArguments,
                       printedBanner);
        
        // 重要
        // 初始化 IoC 容器
        refreshContext(context);
        
        // 在 2.1.4.RELEASE afterRefresh 方法是一个空的方法
        afterRefresh(context, applicationArguments);
        
        // 结束计时
        stopWatch.stop();
        
        // 打印 SpringApplication 启动时间
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        
        // 回调 SpringApplicationRunlisteners 中的 listener.started 方法
        listeners.started(context);
        
        // 重要
        /**
         * 从 IoC 容器中，获取所有 ApplicationRunner 和 CommandLineRunner 的 beans
         * 遍历调用它们的 run 方法
         * (先执行完所有的 ApplicationRunner 再执行 CommandLineRunner 的 run 方法)
         */
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        
        // 回调 SpringApplicationRunlisteners 中的 listener.running 方法
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

`prepareContext` 方法

```java
private void prepareContext(
    ConfigurableApplicationContext context,
	ConfigurableEnvironment environment, 
    SpringApplicationRunListeners listeners,
    ApplicationArguments applicationArguments, Banner printedBanner) {
    
    // 将环境设置保存到 IoC 容器中
    context.setEnvironment(environment);
    
    // 后置处理 IoC 容器
    postProcessApplicationContext(context);
    
    // 回调 ApplicationContextInitializer 的 initialize 方法
    applyInitializers(context);
    
    // 回调 ApplicationListener 的 contextPrepared 方法
    listeners.contextPrepared(context);
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }
    // Add boot specific singleton beans
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    
    // 将命令行中的参数对象注册到 IoC 容器中
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }
    if (beanFactory instanceof DefaultListableBeanFactory) {
        ((DefaultListableBeanFactory) beanFactory)
        .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    // Load the sources
    Set<Object> sources = getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[0]));
    
    // 回调 ApplicationListener 的 contextLoaded 方法
    listeners.contextLoaded(context);
}
```

## 3. 总结

文章开始就提到了 SpringBoot 启动过程中非常重要的 4 个回调类：`ApplicationContextInitializer`、`SpringApplicationRunListener`、`ApplicationRunner` 和 `CommandLineRunner`。

其中，`ApplicationContextInitializer`、`SpringApplicationRunListener` 是需要在 `META-INF/spring.factories` 中配置的。

而 `ApplicationRunner` 和 `CommandLineRunner` 是从 IoC 容器中获取的。

所以，如果我们要自定义回调类，针对于 `ApplicationContextInitializer`、`SpringApplicationRunListener`  是需要在项目中的 `META-INF/spring.factories` 中写成：

```prope
# 以 ApplicationContextInitializer 为例，xxx.xxx.xxx 是自定义类的全限定名
org.springframework.context.ApplicationContextInitializer=xxx.xxx.xxx
```

而针对于 `ApplicationRunner` 或 `CommandLineRunner` ，我们需要在自定义类上加上 `@Component` 注解并且同时实现 `ApplicationRunner` 或 `CommandLineRunner` 接口。