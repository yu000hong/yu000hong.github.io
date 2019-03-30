---
layout: post
title: commons-pool 源码分析：空闲对象驱逐策略
date:   2019-03-27 08:50:00 +0800
tags: [对象池, 连接池, 线程池]
---

与空闲对象驱逐相关的**配置参数**有：

- evictionPolicyClassName
- timeBetweenEvictionRunsMillis
- minEvictableIdleTimeMillis
- softMinEvictableIdleTimeMillis
- numTestsPerEvictionRun
- evictorShutdownTimeoutMillis

与空闲对象驱逐相关的**接口**或**类**有：

- EvictionPolicy
- EvictionConfig
- EvictionTimer
- DefaultEvictionPolicy
- EvictorThreadFactory
- EvictionIterator
- Evictor

## 配置参数

### evictionPolicyClassName

`evictionPolicyClassName`参数指定判断空闲对象是否应该被驱逐的策略类，默认值为：**org.apache.commons.pool2.impl.DefaultEvictionPolicy**，这个类必须实现接口`EvictionPolicy`。这个参数不能设置为null，并且如果设置的类无法加载或没有实现EvictionPolicy接口，都将直接抛出异常。

### timeBetweenEvictionRunsMillis

控制驱逐线程多长时间执行一次，单位毫秒。如果其值为负数或0，那么将不会开启空闲对象驱逐线程。

### minEvictableIdleTimeMillis & softMinEvictableIdleTimeMillis

这两个参数都是用于为空闲对象设置一个空闲时间限制，softMinEvictableIdleTimeMillis为软限制，minEvictableIdleTimeMillis为硬限制。

## Evictor

`Evictor`继承自`TimerTask`，其**run()**方法主要调用了两个方法：

- **evict()**：进行空闲对象的驱逐工作
- **ensureMinIdle()**：确保空闲对象必须不低于`minIdle`，如果低于最小空闲数量，那么调用create()方法生成新对象并加入到空闲队列里

Evictor源码：

{% highlight java %}
class Evictor extends TimerTask {
    @Override
    public void run() {
        final ClassLoader savedClassLoader = Thread.currentThread().getContextClassLoader();
        try {
            //这里是为了解决POOL-161问题，后面会详述该BUG
            if (factoryClassLoader != null) {
                final ClassLoader cl = factoryClassLoader.get();
                if (cl == null) {
                    cancel();
                    return;
                }
                Thread.currentThread().setContextClassLoader(cl);
            }

            // Evict from the pool
            try {
                evict();
            } catch(final Exception e) {
                swallowException(e);
            } catch(final OutOfMemoryError oome) {
                oome.printStackTrace(System.err);
            }
            // Re-create idle instances.
            try {
                ensureMinIdle();
            } catch (final Exception e) {
                swallowException(e);
            }
        } finally {
            // Restore the previous CCL
            Thread.currentThread().setContextClassLoader(savedClassLoader);
        }
    }
}
{% endhighlight %}

Evictor可以关闭么？－可以

我们在实例化GenericObjectPool对象时或调用**setTimeBetweenEvictionRunsMillis()**都会调用**startEvictor(timeBetweenEvictionRunsMillis)**方法开启或关闭Evictor。如果参数`timeBetweenEvictionRunsMillis`小于0将关闭Evictor，如果大于0，那么将开启一个定时器，间隔timeBetweenEvictionRunsMillis毫秒执行一次空闲对象的驱逐工作。

startEvictor()源码：

{% highlight java %}
final void startEvictor(final long delay) {
    synchronized (evictionLock) {
        if (null != evictor) {
            EvictionTimer.cancel(evictor, evictorShutdownTimeoutMillis, TimeUnit.MILLISECONDS);
            evictor = null;
            evictionIterator = null;
        }
        if (delay > 0) {
            evictor = new Evictor();
            EvictionTimer.schedule(evictor, delay, delay);
        }
    }
}
{% endhighlight %}

## EvictorThreadFactory

生成驱逐线程的工厂类，该工厂类的目的是为驱逐线程命名：**commons-pool-evictor-thread**

EvictorThreadFactory源码：

{% highlight java %}
private static class EvictorThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(final Runnable r) {
        final Thread t = new Thread(null, r, "commons-pool-evictor-thread");
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            @Override
            public Void run() {
                t.setContextClassLoader(EvictorThreadFactory.class.getClassLoader());
                return null;
            }
        });
        return t;
    }
}
{% endhighlight %}

## EvictionTimer
EvictionTimer是个工具类，用来启动或停止TimerTask：

{% highlight java %}
static synchronized void schedule(final Runnable task, final long delay, final long period) {
    if (null == executor) {
        executor = new ScheduledThreadPoolExecutor(1, new EvictorThreadFactory());
    }
    usageCount++;
    executor.scheduleWithFixedDelay(task, delay, period, TimeUnit.MILLISECONDS);
}
static synchronized void cancel(final TimerTask task, final long timeout, final TimeUnit unit) {
    task.cancel();
    usageCount--;
    if (usageCount == 0) {
        executor.shutdown();
        try {
            executor.awaitTermination(timeout, unit);
        } catch (final InterruptedException e) {
            // Swallow
            // Significant API changes would be required to propagate this
        }
        executor.setCorePoolSize(0);
        executor = null;
    }
}
{% endhighlight %}

## EvictionIterator

EvictionIterator迭代器实现了接口`Iterator<PooledObject<T>>`，用来迭代`GenericObjectPool<T>`中的空闲对象，根据配置`lifo`值的不同迭代器迭代的顺序不同。

## EvictionConfig

`EvictionConfig`是一个实体类，包含了空闲对象驱逐相关的三个配置参数：

- minEvictableIdleTimeMillis
- softMinEvictableIdleTimeMillis
- minIdle

EvictionConfig把这三个配置项单独提出来的目的是配合`EvictionPolicy`接口实用。

## EvictionPolicy

EvictionPolicy源码：

{% highlight java %}
public interface EvictionPolicy<T> {

    boolean evict(EvictionConfig config, PooledObject<T> underTest, int idleCount);
}
{% endhighlight %}

空闲对象驱逐策略，目前commons-pool提供了一个默认实现`DefaultEvictionPolicy`，我们看看源码：

{% highlight java %}
public class DefaultEvictionPolicy<T> implements EvictionPolicy<T> {
    @Override
    public boolean evict(final EvictionConfig config, final PooledObject<T> underTest, final int idleCount) {
        if ((config.getIdleSoftEvictTime() < underTest.getIdleTimeMillis() &&
                config.getMinIdle() < idleCount) ||
                config.getIdleEvictTime() < underTest.getIdleTimeMillis()) {
            return true;
        }
        return false;
    }
}
{% endhighlight %}

默认情况下，我们是使用DefaultEvictionPolicy做为默认驱逐策略。同时，我们也可以自己实现驱逐策略，进行驱逐相关的定制化。驱逐策略配置参数为：`evictionPolicyClassName`




