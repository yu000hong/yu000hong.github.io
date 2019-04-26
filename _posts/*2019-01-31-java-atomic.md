---
layout: post
title:  Java Atomic 工具包
date:   2019-01-31 15:20:00 +0800
tags: [JAVA, 并发]
---

包名：java.util.concurrent.atomic

文档：<https://docs.oracle.com/javase/8/docs/api/>

> A small toolkit of classes that support lock-free thread-safe programming on single variables.

一共有16个类：

类名 | 描述
:-- | --
**Boolean** | 
`AtomicBoolean` | A boolean value that may be updated atomically.
**Integer** | 
`AtomicInteger` | An int value that may be updated atomically.
`AtomicIntegerArray` | An int array in which elements may be updated atomically.
`AtomicIntegerFieldUpdater<T>` | A reflection-based utility that enables atomic updates to designated volatile int fields of designated classes.
**Long** | 
`AtomicLong` | A long value that may be updated atomically.
`AtomicLongArray` | A long array in which elements may be updated atomically.
`AtomicLongFieldUpdater<T>` | A reflection-based utility that enables atomic updates to designated volatile long fields of designated classes.
**Reference** | 
`AtomicReference<V>` | An object reference that may be updated atomically.
`AtomicReferenceArray<E>` | An array of object references in which elements may be updated atomically.
`AtomicReferenceFieldUpdater<T,V>` | A reflection-based utility that enables atomic updates to designated volatile reference fields of designated classes.
`AtomicMarkableReference<V>` | An AtomicMarkableReference maintains an object reference along with a mark bit, that can be updated atomically.
`AtomicStampedReference<V>` | An AtomicStampedReference maintains an object reference along with an integer "stamp", that can be updated atomically.
**Striped64** | 
`DoubleAccumulator` | One or more variables that together maintain a running double value updated using a supplied function.
`DoubleAdder` | One or more variables that together maintain an initially zero double sum.
`LongAccumulator` | One or more variables that together maintain a running long value updated using a supplied function.
`LongAdder` | One or more variables that together maintain an initially zero long sum.

