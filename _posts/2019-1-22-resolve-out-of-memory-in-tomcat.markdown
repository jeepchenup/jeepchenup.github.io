---
layout : post
title : "解决：Tomcat 热部署时抛出 OutOfMemoryError"
categories: Server
tags: Tomcat
author : ABei
---
* content
{:toc}
最近一直专注于自己的项目开发，期间也发现一些平常工作中不常遇上的问题。



## 情景描述

问题是这样的，Tomcat 能够正常启动，但是修改几次文件之后，在接下来的热部署阶段，经常抛出 **OutOfMemoryError** 这个异常：

```java
java.lang.OutOfMemoryError: PermGen space
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:620)
    at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:124)
```

异常指出了问题出现在永久代，第一个排查的区域就是项目中是否存在创建了大对象的地方。经过一番检查，发现这种情况不存在。接着，我就想是不是加大 Tomcat 内的永久代的内存空间。于是，我就直接上 Google 查询相关的信息，发现这个问题是比较常见的问题。

## 解决方案

默认情况下，Tomcat 为正在运行的进程分配了很小的 PermGen 内存。为了修复这个问题，只需要增加下面的 Java VM 参数就能够增加 PermGen 内存。

```java
-XX:PermSize<size> // 设置 PermGen 初始内存。
-XX:MaxPermSize<size> // 设置 PermGen 最大的内存。
```

既然，知道了增加的参数，那么应该增加在什么地方呢？

### Windows

在 Windows 中，Tomcat 是通过 `catalina.bat` 来启动的。查看 `catalina.bat` 内部代码，你将会发现 `catalina.bat` 总是去检查 `setenv.bat` 文件是否存在，如果存在这个文件，那么将执行这个文件来设置环境变量。

`{$tomcat-folder}\bin\catalina.bat`：

```bash
//...
rem Get standard environment variables
if not exist "%CATALINA_BASE%\bin\setenv.bat" goto checkSetenvHome
call "%CATALINA_BASE%\bin\setenv.bat"
goto setenvDone
:checkSetenvHome
if exist "%CATALINA_HOME%\bin\setenv.bat" call "%CATALINA_HOME%\bin\setenv.bat"
:setenvDone
//...
```

> 注意：`setenv.bat` 文件是需要手动创建的。

放置位置：`${tomcat-folder}\bin\setenv.bat`
文件代码：

```bash
set JAVA_OPTS=-Dfile.encoding=UTF-8 -Xms128m -Xmx1024m -XX:PermSize=64m -XX:MaxPermSize=256m
```

需要重启 Tomcat 才会生效。

### Linux

在 Linux 中，也是相同处理，只是 Tomcat 使用 `catalina.sh` 和 `setenv.sh` 这两个文件来代替。

`catalina.sh` 代码中是一直在检查 `setenv.sh` 文件是否存在。

```shell
//...
# Ensure that any user defined CLASSPATH variables are not used on startup,
# but allow them to be specified in setenv.sh, in rare case when it is needed.
CLASSPATH=

if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
  . "$CATALINA_BASE/bin/setenv.sh"
elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
  . "$CATALINA_HOME/bin/setenv.sh"
fi
//...
```

手动创建 `setenv.sh` 并将其放置于 `${tomcat-folder}\bin\` 目录下。

```shell
export JAVA_OPTS="-Dfile.encoding=UTF-8 -Xms128m -Xmx1024m -XX:PermSize=64m -XX:MaxPermSize=256m"
```

> 注意：这里设置的 PermGen 内存大小只是一个事例，具体大小还是要根据你的项目的具体情况而定。