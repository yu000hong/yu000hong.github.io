---
layout: post
title: commons-pool 源码分析（一）
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

commons-pool比较重要的几个接口：

- PooledObject
- ObjectPool
- PooledObjectFactory
- KeyedObjectPool
- KeyedPooledObjectFactory

## PooledObjectState

池对象状态枚举，有如下状态：

- IDLE
- ALLOCATED
- EVICTION
- EVICTION_RETURN_TO_HEAD
- VALIDATION
- VALIDATION_PREALLOCATED
- VALIDATION_RETURN_TO_HEAD
- INVALID
- ABANDONED
- RETURNING

## PooledObject

所有放入对象池中的对象最终都会被包装为PoolObject，这个包装类提供了一些辅助信息（如对象创建时间、Idle时间、Active时间、上次借出时间、上次归还时间等），便于我们对对象生命周期的管理。

接口方法：

{% highlight java %}
public interface PooledObject<T> {
    T getObject();
    PooledObjectState getState();
    long getCreateTime();
    long getActiveTimeMillis();
    long getIdleTimeMillis();
    long getLastBorrowTime();
    long getLastReturnTime();
    long getLastUsedTime();
    boolean startEvictionTest();
    boolean endEvictionTest(Deque<PooledObject<T>> idleQueue);
    boolean allocate();
    boolean deallocate();
    void invalidate();
    void setLogAbandoned(boolean logAbandoned);
    void use();
    void markAbandoned();
    void markReturning();
}
{% endhighlight %}

PooledOjbect
     |
DefaultPooledObject
     |
PooledSoftReference



## ObjectPool

ObjectPool有三种实现：

- GenericObjectPool
- SoftReferenceObjectPool
- ProxiedObjectPool

ObjectPool接口方法：

{% highlight java %}
public interface ObjectPool<T> {
    T borrowObject();
    void returnObject(T obj);
    void invalidateObject(T obj);
    void addObject();
    int getNumIdle();
    int getNumActive();
    void clear();
    void close();
}
{% endhighlight %}

## GenericObjectPool





