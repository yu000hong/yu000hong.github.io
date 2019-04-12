---
layout: post
title: commons-pool 源码分析：PooledObjectFactory
date:   2019-04-11 10:00:00 +0800
tags: [对象池, 连接池, 线程池]
---

<!-- more -->

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

当对象池对象不够用的时候，需要调用此方法创建一个新对象放入到池子里。

**destroyObject()**

不管什么原因（丢弃、驱逐、检验失败、异常等），当对象需要销毁时，都会调用此方法进行销毁处理。

**validateObject()**

对象进行检验时，都会调用此方法。

一共4种检验：

- testOnCreate
- testOnBorrow
- testOnReturn
- testWhileIdle

**activateObject()**

当对象借出后，必须调用此方法激活对象。

**passivateObject()**

当对象归还时，必须调用此方法钝化对象。

## 对象池使用

我们要基于**commons-pool**构建自己的线程池、连接池等，只需要实现`PooledObjectFactory`接口即可，如下代码为一个简单的连接池实现：

{% highlight java %}
public class ConnectionFactory implements PooledObjectFactory<Connection> {
    public PooledObject<Connection> makeObject() throws Exception {
        Connection conn = new Connection();
        conn.open();
        return new DefaultPooledObject(conn);
    }
    public void destroyObject(PooledObject<T> p) throws Exception {
        Connection conn = p.getObject();
        conn.close();
    }
    public boolean validateObject(PooledObject<T> p) {
        Connection conn = p.getObject();
        try {
            conn.ping();
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    public void activateObject(PooledObject<T> p) throws Exception {
        //do nothing
    }
    public void passivateObject(PooledObject<T> p) throws Exception {
        //do nothing
    }
}

public class Demo {
    public static void main(String[] args) {
        ConnectionFactory factory = new ConnectionFactory();
        GenericObjectPoolConfig config = new GenericObjectPoolConfig();
        GenericObjectPool<Connection> pool = new GenericObjectPool<Connection>(factory, config);
        Connection conn = pool.borrowObject();
        //处理业务逻辑
        pool.use(conn);//对象租借续期，避免被丢弃
        //处理业务逻辑
        pool.returnObject(conn);
        //清空对象，退出程序
        pool.clear();
    }
}
{% endhighlight %}









