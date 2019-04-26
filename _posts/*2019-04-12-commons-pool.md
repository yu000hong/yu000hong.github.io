---
layout: post
title: commons-pool 配置参数
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

<!-- more -->

## 配置类

配置参数涉及三个类：

```
                BaseObjectPoolConfig
                        ｜
            －－－－－－－－－－－－－－－－
            ｜                         ｜
GenericObjectPoolConfig     GenericKeyedObjectPoolConfig      

```

## 配置参数

commons pool 一共涉及23个配置参数：

- maxTotal
- maxIdle
- minIdle
- maxTotalPerKey
- maxIdlePerKey
- minIdlePerKey
- lifo
- fairness
- blockWhenExhausted
- maxWaitMillis
- testOnCreate
- testOnBorrow
- testOnReturn
- testWhileIdle
- evictionPolicyClassName
- minEvictableIdleTimeMillis
- softMinEvictableIdleTimeMillis
- numTestsPerEvictionRun
- evictorShutdownTimeoutMillis
- jmxEnabled
- jmxNameBase
- JmxNamePrefix

### maxTotal

最大总对象数量，默认值8

### maxIdle

最大空闲对象数量，默认值8


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
jmxNameBase||org.apache.commons.pool2:type=GenericObjectPool,name=
jmxNamePrefix||pool










