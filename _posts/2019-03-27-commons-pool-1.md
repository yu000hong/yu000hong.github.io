---
layout: post
title: commons-pool 源码分析（一）
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

## PooledObjectState

PooledObjectState是一个枚举类，表示对象在对象池的当前状态。

所有的池对象最终都会被包装为一个`PooledObject`对象，所有放入对象池中的对象最终都会被包装为PoolObject，这个包装类提供了一些辅助信息（如对象创建时间、Idle时间、Active时间、上次借出时间、上次归还时间等），便于我们对对象生命周期的管理。

PooledObject对象还会维护一个枚举状态，即PooledObjectState。

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

## 对象池配置参数

配置参数涉及三个类：

```
                BaseObjectPoolConfig
                        ｜
            －－－－－－－－－－－－－－－－
            ｜                         ｜
GenericObjectPoolConfig     GenericKeyedObjectPoolConfig      

```

- maxTotal/maxTotalPerKey：最大总对象数量
- maxIdle/getMaxIdlePerKey：
- minIdle/minIdlePerKey：
- lifo
- fairness
- maxWaitMillis
- minEvictableIdleTimeMillis
- softMinEvictableIdleTimeMillis
- numTestsPerEvictionRun
- evictorShutdownTimeoutMillis
- testOnCreate
- testOnBorrow
- testOnReturn
- testWhileIdle
- timeBetweenEvictionRunsMillis
- evictionPolicyClassName
- blockWhenExhausted
- jmxEnabled
- jmxNameBase
- jmxNamePrefix


--|--|--
参数|含义|默认值
maxTotal|最大总对象数量|8
maxIdle|最大空闲对象数量|8







