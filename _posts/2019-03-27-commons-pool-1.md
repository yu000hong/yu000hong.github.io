---
layout: post
title: commons-pool 源码分析（一）
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

## PooledObjectState

`PooledObjectState`是一个枚举类，表示对象在对象池的当前状态。

所有的池对象最终都会被包装为一个`PooledObject`对象，所有放入对象池中的对象最终都会被包装为PooledObject，这个包装类提供了一些辅助信息（如对象创建时间、Idle时间、Active时间、上次借出时间、上次归还时间等），便于我们对对象生命周期的管理。

PooledObject对象还会维护一个枚举状态，即：`PooledObjectState`。

池对象状态枚举，有如下状态：

- IDLE：对象处于空闲状态
- ALLOCATED：表示对象已经被借出，正在使用
- EVICTION：对象正在进行驱逐检测
- EVICTION_RETURN_TO_HEAD：在对象处于驱逐检测过程中时被强行allocate()
- VALIDATION
- VALIDATION_PREALLOCATED
- VALIDATION_RETURN_TO_HEAD
- INVALID：经过驱逐检测发现，这个空闲对象需要被驱逐，将被置为INVALID
- ABANDONED：
- RETURNING：将对象归还到对象池中时，对象的一个临时状态

## 状态转移图

## 对象池配置参数

配置参数涉及三个类：

```
                BaseObjectPoolConfig
                        ｜
            －－－－－－－－－－－－－－－－
            ｜                         ｜
GenericObjectPoolConfig     GenericKeyedObjectPoolConfig      

```

--|--|--
参数|含义|默认值
maxTotal|最大总对象数量|8
maxIdle|最大空闲对象数量|8
minIdle|确保最少空闲对象数量|0
maxTotalPerKey||8
maxIdlePerKey||8
minIdlePerKey||0
lifo|对象池中的对象是否确保后进先出的顺序|true
fairness|是否确保公平地从对象池中取出对象|false
blockWhenExhausted|如果对象池为空，是否阻塞等待|true
maxWaitMillis|最大阻塞等待时间，单位毫秒|-1
testOnCreate|当对象创建时是否进行有效性校验|false
testOnBorrow|当对象借出时是否进行有效性校验|false
testOnReturn|当对象归还时是否进行有效性校验|false
testWhileIdle|当对象空闲时是否进行有效性校验|false
evictionPolicyClassName||org.apache.commons.pool2.impl.DefaultEvictionPolicy
minEvictableIdleTimeMillis||1000*60*30
softMinEvictableIdleTimeMillis||-1
numTestsPerEvictionRun|在每个空闲对象驱逐线程运行过程中中进行检查的对象个数|3
evictorShutdownTimeoutMillis||1000*10
timeBetweenEvictionRunsMillis||-1
jmxEnabled||true
jmxNameBase||null
jmxNamePrefix||pool










