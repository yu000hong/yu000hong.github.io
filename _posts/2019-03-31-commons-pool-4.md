---
layout: post
title: commons-pool 源码分析：AbandonedConfig
date:   2019-03-31 15:20:00 +0800
tags: [对象池, 连接池, 线程池]
---

对象池中的对象被借出后，因为程序BUG、借出线程休眠或者借出对象本身在处理时长时间等待等原因，有可能会长时间持有对象而不归还，如果大量线程都借出对象而不归还的话，会导致对象池被借空而无法应对更多的对象借出请求。出现这种不可控的情况的话，对象池有没有其他办法呢？

> 答案就在`AbandonedConfig`里面，`AbandonedConfig`配置一些参数，用来控制对象在借出后长时间不归还的情况下进行丢弃（Abandon）操作。

那如果借出对象的线程确实需要一些长时间的操作，怎么能保证借出对象不会被对象池丢弃呢？

> 这种情况就需要调用`GernericObjectPool.use(obj)`方法对借出对象进行续期，这个方法最终会调用`DefaultPooledObject.use()`方法，更新池对象的上次使用时间lastUseTime为当前时间。

## AbandonedConfig

`AbandonedConfig`配置类提供了如下配置：

--|--
配置参数|说明|默认值
removeAbandonedOnBorrow|是否在借出对象时进行丢弃检测操作|false
removeAbandonedOnMaintenance|是否在执行Evictor线程中进行丢弃检测操作|false
removeAbandonedTimeout|借出对象在多长时间未使用后即可进行丢弃，单位秒|300
logAbandoned|是否打印借出对象的调用堆栈信息|false
logWriter|堆栈信息打印的最终输出地|标准输出`new PrintWriter(System.out)`
requireFullStackTrace|是否需要打印全量的堆栈信息|true
useUsageTracking|TODO|false

我们如何设置AbandonedConfig呢？

> 在实例化`GenericObjectPool`对象时，作为构造方法参数传入即可，或者调用其`setAbandonedConfig()`方法。如果abandonedConfig为null，对象池将不会对借出对象进行时间检测做丢弃处理。

{% highlight java %}
public GenericObjectPool(final PooledObjectFactory<T> factory,
        final GenericObjectPoolConfig config, final AbandonedConfig abandonedConfig) {
    this(factory, config);
    setAbandonedConfig(abandonedConfig);
}
void setAbandonedConfig(final AbandonedConfig abandonedConfig);
{% endhighlight %}

## 丢弃检测

丢弃检测操作由`removeAbandoned()`方法完成，这个操作比较费时，它会遍历所有对象，以找出需要丢弃的对象，然后进行丢弃处理。判断对象是否可被丢弃的条件有：

- 对象处于**ALLOCATED**状态
- 对象上次使用时间距今已经超过设置的`removeAbandonedTimeout`

代码如下：

{% highlight java %}
private void removeAbandoned(final AbandonedConfig ac) {
    final long now = System.currentTimeMillis();
    final long timeout = now - (ac.getRemoveAbandonedTimeout() * 1000L);
    final ArrayList<PooledObject<T>> remove = new ArrayList<>();
    final Iterator<PooledObject<T>> it = allObjects.values().iterator();
    while (it.hasNext()) {
        final PooledObject<T> pooledObject = it.next();
        synchronized (pooledObject) {
            //判断对象是否可被丢弃
            if (pooledObject.getState() == PooledObjectState.ALLOCATED &&
                    pooledObject.getLastUsedTime() <= timeout) {
                pooledObject.markAbandoned();
                remove.add(pooledObject);
            }
        }
    }
    final Iterator<PooledObject<T>> itr = remove.iterator();
    while (itr.hasNext()) {
        final PooledObject<T> pooledObject = itr.next();
        if (ac.getLogAbandoned()) {
            pooledObject.printStackTrace(ac.getLogWriter());
        }
        try {
            invalidateObject(pooledObject.getObject());
        } catch (final Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

## removeAbandonedOnBorrow

在调用`borrowObject()`方法借出对象的时候进行丢弃检测，代码如下：

{% highlight java %}
public T borrowObject(final long borrowMaxWaitMillis) throws Exception {
    assertOpen();
    final AbandonedConfig ac = this.abandonedConfig;
    if (ac != null && ac.getRemoveAbandonedOnBorrow() &&
            (getNumIdle() < 2) &&
            (getNumActive() > getMaxTotal() - 3) ) {
        removeAbandoned(ac);
    }
    ...
    ...
}
{% endhighlight %}

即使设置`removeAbandonedOnBorrow`为true，也必须满足另外两个条件，才会进行丢弃检测操作：

- 空闲对象数量小于2
- 活动对象数量大于maxTotal-3

为什么会有这样的限制呢？因为丢弃检测操作`removeAbandoned(ac)`是一个比较费时的操作，它会遍历池中的所有对象。

## removeAbandonedOnMaintenance

在调用`evict()`方法驱逐对象的时候进行丢弃检测，代码如下：

{% highlight java %}
public void evict() throws Exception {
    assertOpen();
    ...
    ...
    final AbandonedConfig ac = this.abandonedConfig;
    if (ac != null && ac.getRemoveAbandonedOnMaintenance()) {
        removeAbandoned(ac);
    }
}
{% endhighlight %}

## requireFullStackTrace

TODO

## useUsageTracking

TODO



