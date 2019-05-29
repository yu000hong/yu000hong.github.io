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

### 内存溢出

**堆溢出**

Java堆用于存储对象实例，只要不断地创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，对象数量达到最大堆容量限制，则发生溢出。

报错：

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid7768.hprof ...
Heap dump file created [27987840 bytes in 0.142 secs]
```

**栈溢出**

HotSpot不区分虚拟机栈和本地方法栈，栈容量只能由-Xss参数设定。

- StackOverFlow：线程申请的栈深度超过允许的最大深度
  ```
  Exception in thread "main" java.lang.StackOverflowError
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    ......
  ```
- OutOfMemoryError： 多线程下的栈溢出，与栈空间是否足够大并不存在任何联系。为每个线程的栈分配的内存越大（参数-Xss），那么可以建立的线程数量就越少，建立线程时就越容易把剩下的内存耗尽，越容易内存溢出。在这种情况下，如果不能减少线程数目或者更换64位虚拟机时，减少最大堆和减少栈容量能够换区更多的线程。

**方法区溢出**

方法区用于存放Class的相关信息，如果运行时产生大量的类去填满方法区，就可能发生方法区的内存溢出。 例如主流框架Spring、Hibernate对大量的类进行增强时，利用CGLib字节码生成动态类；大量JSP或动态JSP(JSP第一次运行时需要编译为Java类）。

```
Caused by: java.lang.OutOfMemoryError: PermGen space
    at java.lang.ClassLoader.defineClass1(Native Method)
    ......
```

**常量池溢出**

String.intern()是一个Native方法，它的作用是：如果运行时常量池中已经包含一个等于此String对象内容的字符串，则返回常量池中该字符串的引用；如果没有，则在常量池中创建与此String内容相同的字符串，并返回常量池中创建的字符串的引用。JDK7的intern()方法的实现有所不同，当常量池中没有该字符串时，不再是在常量池中创建与此String内容相同的字符串，而改为在常量池中记录堆中首次出现的该字符串的引用，并返回该引用。

```
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
    at java.lang.String.intern(Native Method)
    ......
```

**直接内存溢出**

Java虚拟机可以通过参数-XX:MaxDirectMemorySize设定本机直接内存可用大小，如果不指定，则默认与java堆内存大小相同。JDK中可以通过反射获取Unsafe类(Unsafe的getUnsafe()方法只有启动类加载器Bootstrap才能返回实例)直接操作本机直接内存。通过使用-XX:MaxDirectMemorySize=10M，限制最大可使用的本机直接内存大小为10MB。

```
Exception in thread "main" java.lang.OutOfMemoryError
    at sun.misc.Unsafe.allocateMemory(Native Method)
    ......
```

