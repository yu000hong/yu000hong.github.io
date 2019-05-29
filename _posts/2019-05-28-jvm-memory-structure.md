---
layout: post
title: JVM 内存结构
date: 2019-05-28 17:00:00 +0800
tags: [JVM, JMM]
---

所有的Java开发人员可能会遇到这样的困惑？我该为堆内存设置多大空间呢？OutOfMemoryError的异常到底涉及到运行时数据的哪块区域？该怎么解决呢？其实如果你经常解决服务器性能问题，那么这些问题就会变的非常常见，了解JVM内存也是为了服务器出现性能问题的时候可以快速的了解那块的内存区域出现问题，以便于快速的解决生产故障。

<!-- more -->

### JVM内存结构

![](/static/image/201905/jvm-memory-structure.png)

JVM内存结构主要有三大块：

- 堆内存
- 方法区
- 栈

**堆内存**

堆内存是JVM中最大的一块，由新生代和老年代组成；

而新生代内存又被分成三部分，Eden空间、From Survivor空间、To Survivor空间，默认情况下新生代按照`8:1:1`的比例来分配；

**方法区**

方法区存储类信息、常量、静态变量等数据，是线程共享的区域；

为与Java堆区分，方法区还有一个别名Non-Heap(非堆)；

**栈**

栈又分为Java虚拟机栈和本地方法栈，主要用于方法的执行。

### 控制参数

**-Xms**：设置堆的最小空间大小

**-Xmx**：设置堆的最大空间大小

**-Xmn**：设置新生代的空间大小（`-Xmn1g`效果等同于`-XX:NewSize=-XX:MaxNewSize=1g`）

**-XX:NewSize**：设置新生代最小空间大小

**-XX:MaxNewSize**：设置新生代最大空间大小

**-XX:PermSize**：设置永久代最小空间大小

**-XX:MaxPermSize**：设置永久代最大空间大小

**-XX:ThreadStackSize**：设置每个线程的堆栈大小

**-Xss**：-Xss是-XX:ThreadStackSize的别名，但`-XX`选项不稳定，有可能在以后的版本不支持

**-XX:MaxTenuringThreshold**：控制对象能经历多少次Minor GC才晋升到老年代，默认值是15

**-XX:NewRatio**：新生代和老年代的比值，4表示“新生代:老年代=1:4”，即新生代占堆的1/5

**-XX:SurvivorRatio**：设置两个Survivor和Eden的比值，8表示“两个Survivor:Eden=2:8”，即一个Survivor占新生代的1/10

