---
layout: post
title: commons-pool 对象池技术
date:   2019-03-26 14:30:00 +0800
tags: [对象池, 连接池, 线程池]
---

**为什么需要对象池？**

- 降低资源消耗：资源对象的创建和销毁开销较大，通过重复利用资源来降低开销；

- 提高响应速度：当需要资源时，可以直接从对象池中直接获取，快速返回资源；

- 资源可管理性：资源是稀缺的，如果无限制的创建，不仅会消耗系统资源，还会降低系统稳定性。把资源放在对象池中进行统一管理，可以进行监控和调优。

`commons-pool`项目是[Apache Commons](http://commons.apache.org/)项目下面的一个子项目，该项目旨在提供一个通用对象池技术，为我们实现各种连接池、线程池提供了基础实现，它能尽量地将使用方和对象池本身做一个很好的解耦，我们只需要实现一些简单的基本逻辑即可（实现接口`PoolableObjectFactory`）。

我们熟知的[Commons DBCP](http://commons.apache.org/proper/commons-dbcp/)数据库连接池和[Jedis](https://github.com/xetorthio/jedis) Redis的Java客户端底层就是使用的**commons-pool**实现的，DBCP和Jedis很多的参数配置是一样的，都源自commons-pool，这些参数其实就是用来控制底层对象池的行为的，如：

- maxIdle
- minIdle
- maxTotal
- testOnCreate
- testOnBorrow
- testOnReturn
- testWhileIdle
- maxWaitMillis
- lifo
- ...

阿里开源的数据库连接池[Druid](https://github.com/alibaba/druid)虽然底层没有采用commons-pool作为对象池实现，但是基本也沿用了这些配置名称，所以我们在配置这些连接池的时候看到这些参数名称就能很快明白参数的意义了。

## Jedis

主要实现类：`JedisFactory`

下面看下主要实现代码：

{% highlight java %}
class JedisFactory implements PooledObjectFactory<Jedis> {

  //......省略了一些代码

  @Override
  public void activateObject(PooledObject<Jedis> pooledJedis) throws Exception {
    final BinaryJedis jedis = pooledJedis.getObject();
    if (jedis.getDB() != database) {
      jedis.select(database);
    }
  }

  @Override
  public void destroyObject(PooledObject<Jedis> pooledJedis) throws Exception {
    final BinaryJedis jedis = pooledJedis.getObject();
    if (jedis.isConnected()) {
      try {
        try {
          jedis.quit();
        } catch (Exception e) {
        }
        jedis.disconnect();
      } catch (Exception e) {
      }
    }
  }

  @Override
  public PooledObject<Jedis> makeObject() throws Exception {
    final HostAndPort hostAndPort = this.hostAndPort.get();
    final Jedis jedis = new Jedis(hostAndPort.getHost(), hostAndPort.getPort(),
        connectionTimeout, soTimeout, ssl, sslSocketFactory, sslParameters, hostnameVerifier);
    try {
      jedis.connect();
      if (password != null) {
        jedis.auth(password);
      }
      if (database != 0) {
        jedis.select(database);
      }
      if (clientName != null) {
        jedis.clientSetname(clientName);
      }
    } catch (JedisException je) {
      jedis.close();
      throw je;
    }
    return new DefaultPooledObject<Jedis>(jedis);
  }

  @Override
  public void passivateObject(PooledObject<Jedis> pooledJedis) throws Exception {
    // TODO maybe should select db 0? Not sure right now.
  }

  @Override
  public boolean validateObject(PooledObject<Jedis> pooledJedis) {
    final BinaryJedis jedis = pooledJedis.getObject();
    try {
      HostAndPort hostAndPort = this.hostAndPort.get();
      String connectionHost = jedis.getClient().getHost();
      int connectionPort = jedis.getClient().getPort();
      return hostAndPort.getHost().equals(connectionHost)
          && hostAndPort.getPort() == connectionPort && jedis.isConnected()
          && jedis.ping().equals("PONG");
    } catch (final Exception e) {
      return false;
    }
  }

}
{% endhighlight %}

Jedis使用示例：

{% highlight java %}
JedisPoolConfig poolConfig = new JedisPoolConfig();
poolConfig.setMaxTotal(128);
poolConfig.setMaxIdle(128);
poolConfig.setMinIdle(16);
poolConfig.setTestOnBorrow(true);
poolConfig.setTestOnReturn(true);
poolConfig.setTestWhileIdle(true);
poolConfig.setMinEvictableIdleTimeMillis(Duration.ofSeconds(60).toMillis());
poolConfig.setTimeBetweenEvictionRunsMillis(Duration.ofSeconds(30).toMillis());
poolConfig.setNumTestsPerEvictionRun(3);
poolConfig.setBlockWhenExhausted(true);

JedisPool jedisPool = new JedisPool(poolConfig, "localhost");
try (Jedis jedis = jedisPool.getResource()) {
    // do operations with jedis resource
}
{% endhighlight %}

使用Jedis时，我们最重要的是实例化JedisPool，通过JedisPool我们就可以使用Redis的连接池了。我们在new JedisPool的时候，会自动生成一个JedisFactory实例（实现了PooledObjectFactory接口），这个实例就是我们使用commons-pool的桥梁了。通过代码可以看出，使用commons-pool还是挺简单的！

下面贴出JedisPool构造方法代码：

{% highlight java %}
public JedisPool(final GenericObjectPoolConfig poolConfig, final URI uri,
    final int connectionTimeout, final int soTimeout) {
    super(poolConfig, new JedisFactory(uri, connectionTimeout, soTimeout, null));
}
{% endhighlight %}

## Commons DBCP

DBCP的代码要复杂一些，留待后面分析。

## 参考资料

[Apache Commons Pool 官网](http://commons.apache.org/proper/commons-pool/)

[Apache Commons Pool源码分析](https://www.jianshu.com/p/b49452fb3a67)

[对象池common-pool2源码分析之对象状态](https://my.oschina.net/u/657390/blog/659502)

[如何设计一个连接池：commons-pool2源码分析](https://throwsnew.com/2017/06/12/commons-pool/)



