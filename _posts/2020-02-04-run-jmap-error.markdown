---
layout: post
title: "执行jmap执行异常"
author: ABei
categories : Java
tags: JVM Commands
---
* content
{:toc}

今天遇到执行`jmap`指令出现如下异常：




```txt
C:\Program Files\Java\jdk1.8.0_121\bin>jmap -heap 3460
Attaching to process ID 3460, please wait...
Exception in thread "main" java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at sun.tools.jmap.JMap.runTool(JMap.java:201)
        at sun.tools.jmap.JMap.main(JMap.java:130)
Caused by: java.lang.InternalError: void* type hasn't been seen when parsing int*
        at sun.jvm.hotspot.HotSpotTypeDataBase.recursiveCreateBasicPointerType(HotSpotTypeDataBase.java:721)
        at sun.jvm.hotspot.HotSpotTypeDataBase.lookupType(HotSpotTypeDataBase.java:134)
        at sun.jvm.hotspot.HotSpotTypeDataBase.lookupOrCreateClass(HotSpotTypeDataBase.java:631)
        at sun.jvm.hotspot.HotSpotTypeDataBase.createType(HotSpotTypeDataBase.java:751)
        at sun.jvm.hotspot.HotSpotTypeDataBase.readVMTypes(HotSpotTypeDataBase.java:195)
        at sun.jvm.hotspot.HotSpotTypeDataBase.<init>(HotSpotTypeDataBase.java:89)
        at sun.jvm.hotspot.HotSpotAgent.setupVM(HotSpotAgent.java:391)
        at sun.jvm.hotspot.HotSpotAgent.go(HotSpotAgent.java:305)
        at sun.jvm.hotspot.HotSpotAgent.attach(HotSpotAgent.java:140)
        at sun.jvm.hotspot.tools.Tool.start(Tool.java:185)
        at sun.jvm.hotspot.tools.Tool.execute(Tool.java:118)
        at sun.jvm.hotspot.tools.HeapSummary.main(HeapSummary.java:49)
        ... 6 more
```

经过一番查询，发现是JDK版本不一致造成的！所以，我先确认一下当前Java进程所使用的JDK版本。

执行指令：`jcmd 3460 VM.version`，输出如下结果：

```txt
C:\Program Files\Java\jdk1.8.0_121\bin>jcmd 3460 VM.version
3460:
Java HotSpot(TM) 64-Bit Server VM version 24.80-b11
JDK 7.0_80
```

而当前我的所在的目录为JDK1.8的。所以，换个路径看看：

```txt
C:\Program Files\Java\jdk1.7.0_80\bin>jmap -heap 3460
Attaching to process ID 3460, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.80-b11

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 0
   MaxHeapFreeRatio = 100
   MaxHeapSize      = 6413090816 (6116.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 21757952 (20.75MB)
   MaxPermSize      = 85983232 (82.0MB)
   G1HeapRegionSize = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 101187584 (96.5MB)
   used     = 2023848 (1.9300918579101562MB)
   free     = 99163736 (94.56990814208984MB)
   2.0000951895442034% used
From Space:
   capacity = 16252928 (15.5MB)
   used     = 0 (0.0MB)
   free     = 16252928 (15.5MB)
   0.0% used
To Space:
   capacity = 16252928 (15.5MB)
   used     = 0 (0.0MB)
   free     = 16252928 (15.5MB)
   0.0% used
PS Old Generation
   capacity = 102236160 (97.5MB)
   used     = 753488 (0.7185821533203125MB)
   free     = 101482672 (96.78141784667969MB)
   0.737007336738782% used
PS Perm Generation
   capacity = 22020096 (21.0MB)
   used     = 3971816 (3.7878189086914062MB)
   free     = 18048280 (17.212181091308594MB)
   18.037232898530505% used

2934 interned Strings occupying 241888 bytes.
```