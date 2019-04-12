---
layout: post
title: commons-pool 源码分析：PooledObjectFactory
date:   2019-04-11 10:00:00 +0800
tags: [对象池, 连接池, 线程池]
---

<!-- more -->

## PooledObjectFactory

`PooledObjectFactory`是一个接口，定义如下：

{% highlight java %}
public interface PooledObjectFactory<T> {
    PooledObject<T> makeObject() throws Exception;
    void destroyObject(PooledObject<T> p) throws Exception;
    boolean validateObject(PooledObject<T> p);
    void activateObject(PooledObject<T> p) throws Exception;
    void passivateObject(PooledObject<T> p) throws Exception;
}
{% endhighlight %}

**makeObject()**

当对象池对象不够用的时候，需要调用`makeObject()`方法创建一个新对象放入到池子里。

**destroyObject()**

不管什么原因（丢弃、驱逐、检验失败、异常等），当对象需要销毁时，都会调用`destroyObject()`方法进行销毁处理。

**validateObject()**

对象进行检验时，会调用`validateObject()`方法，包括：**testOnCreate**、**testOnBorrow**、**testOnReturn**、**testWhileIdle**这些检验。

**activateObject()**

当对象借出后，必须调用`activateObject()`对对象进行激活。

**passivateObject()**

当对象归还时，必须调用`passivateObject()`对对象进行钝化。











