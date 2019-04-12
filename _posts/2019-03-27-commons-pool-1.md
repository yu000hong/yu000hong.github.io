---
layout: post
title: commons-pool 源码分析：池对象的生命周期
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

## PooledObjectState

`PooledObjectState`是一个枚举类，表示对象在对象池的当前状态，有如下状态：

- IDLE：对象处于空闲状态
- ALLOCATED：表示对象已经被借出，正在使用
- EVICTION：对象正在进行驱逐检测
- EVICTION_RETURN_TO_HEAD：在对象处于驱逐检测过程中时被强行allocate()
- INVALID：经过驱逐检测发现，这个空闲对象需要被驱逐，将被置为INVALID
- ABANDONED：被丢弃状态
- RETURNING：将对象归还到对象池中时，对象的一个临时状态
- VALIDATION：暂时未使用
- VALIDATION_PREALLOCATED：暂时未使用
- VALIDATION_RETURN_TO_HEAD：暂时未使用

## DefaultPooledObject

所有的池对象最终都会被包装为一个`PooledObject`对象，`PooledObject`是一个接口，默认实现类为`DefaultPooledObject`。`DefaultPooledObject`这个包装类除了维护对象状态外，还提供了一些辅助信息（如对象创建时间、Idle时间、Active时间、上次借出时间、上次归还时间等），便于我们对对象生命周期的管理。

对象生命周期相关的方法有：

- startEvictionTest()
- endEvictionTest()
- allocate()
- deallocate()
- invalidate()
- markAbandoned()
- markReturning()
- use()

**startEvictionTest()**

这个方法在Evictor线程执行`evict()`方法时被调用，开始执行驱逐检测，对象将被置为**EVICTION**状态。

**endEvictionTest()**

这个方法在Evictor线程执行`evict()`方法时被调用，表示完成驱逐检测。如果对象满足驱逐条件时，将被置为**INVALID**状态然后被销毁；如果对象不满足驱逐条件，那么将会被重新置为**IDLE**状态。

**allocate()**

在借出对象（调用borrowObject()方法）时，会调用对象的allocate方法；如果对象当前处于**IDLE**状态，表明对象目前处于空闲状态可以借出，此时对象将被置为**ALLOCATED**状态；如果对象处于**EVICTION**状态，表明对象正处于驱逐检测过程中，对象将被置为**EVICTION_RETURN_TO_HEAD**，然后在完成驱逐检测时该对象将被移到空闲队列的最前端（为什么要移到最前端呢？）。

**deallocate()**

在归还对象（调用returnObject()方法）时，会调用对象的deallocate方法，对象将被重新置为**IDLE**状态。

**invalidate()**

当对象因为各种原因需要被销毁时，会调用该方法，将对象状态置为**INVALID**，同时会从空闲对象列表和全部对象列表中移除对应的对象。

那么，什么时候对象需要销毁呢？

- 调用`PooledObjectFactory.activateObject()`异常时
- 调用`PooledObjectFactory.passivateObject()`异常时
- 调用`PooledObjectFactory.validateObject()`异常或失败时
- 主动调用`ObjectPool.invalidateObject()`方法
- 主动调用`ObjectPool.clear()`方法
- 在`evict()`的过程中，对象需要被驱逐的时候

等等，不是应该还有丢弃操作（Abandon）也会销毁对象么？对，丢弃操作也会销毁对象，但是丢弃操作没有直接进行销毁，而是主动调用了`ObjectPool.invalidateObject()`方法。

**markAbandoned()**

当对象池整体状态满足进行丢弃操作的条件时，对象池会对所有对象进行一次丢弃检测。如果对象需要被丢弃，那么将调用该方法将对象状态置为**ABANDONED**。

**markReturning()**

当归还对象时，将会调用该方法将对象的状态置为**RETURNING**，然后在一系列处理(`factory.validateObject()`,`factory.passivateObject()`)之后对象的最终状态将被置为**IDLE**。

那这里为什么会有一个中间状态呢？因为这里存在并发问题，在归还过程的同时有可能对象正在被丢弃。如果丢弃操作先占用，那么对象将被置为**ABANDONED**，然后**INVALID**，直至销毁；如果归还操作先占用，那么对象将被置为**RETURNING**，然后**IDLE**，最终变成空闲对象。

**use()**

这个方法用于设置对象的`lastUseTIme`，避免对象被丢弃。

## 状态转移图

![](/static/image/201904/commons-pool-states.png)








