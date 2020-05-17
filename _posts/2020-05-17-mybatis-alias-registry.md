---
layout: post
title: 「MyBatis源码分析」类型别名
date:   2020-05-17 23:53:00 +0800
tags: [MyBatis]
---

MyBatis 类型别名可为 Java 类型设置一个缩写名字，它**`仅用于`** XML 配置，意在降低冗余的全限定类名书写。

## 内置别名

MyBatis内置的类型别名有(全部)：

{% highlight java %}
registerAlias("string", String.class);

registerAlias("byte", Byte.class);
registerAlias("long", Long.class);
registerAlias("short", Short.class);
registerAlias("int", Integer.class);
registerAlias("integer", Integer.class);
registerAlias("double", Double.class);
registerAlias("float", Float.class);
registerAlias("boolean", Boolean.class);

registerAlias("byte[]", Byte[].class);
registerAlias("long[]", Long[].class);
registerAlias("short[]", Short[].class);
registerAlias("int[]", Integer[].class);
registerAlias("integer[]", Integer[].class);
registerAlias("double[]", Double[].class);
registerAlias("float[]", Float[].class);
registerAlias("boolean[]", Boolean[].class);

registerAlias("_byte", byte.class);
registerAlias("_long", long.class);
registerAlias("_short", short.class);
registerAlias("_int", int.class);
registerAlias("_integer", int.class);
registerAlias("_double", double.class);
registerAlias("_float", float.class);
registerAlias("_boolean", boolean.class);

registerAlias("_byte[]", byte[].class);
registerAlias("_long[]", long[].class);
registerAlias("_short[]", short[].class);
registerAlias("_int[]", int[].class);
registerAlias("_integer[]", int[].class);
registerAlias("_double[]", double[].class);
registerAlias("_float[]", float[].class);
registerAlias("_boolean[]", boolean[].class);

registerAlias("date", Date.class);
registerAlias("decimal", BigDecimal.class);
registerAlias("bigdecimal", BigDecimal.class);
registerAlias("biginteger", BigInteger.class);
registerAlias("object", Object.class);

registerAlias("date[]", Date[].class);
registerAlias("decimal[]", BigDecimal[].class);
registerAlias("bigdecimal[]", BigDecimal[].class);
registerAlias("biginteger[]", BigInteger[].class);
registerAlias("object[]", Object[].class);

registerAlias("map", Map.class);
registerAlias("hashmap", HashMap.class);
registerAlias("list", List.class);
registerAlias("arraylist", ArrayList.class);
registerAlias("collection", Collection.class);
registerAlias("iterator", Iterator.class);

registerAlias("ResultSet", ResultSet.class);
{% endhighlight %}

## 别名配置

{% highlight xml %}
<typeAliases>
    <package name="com.yu000hong.domain"/>
    <typeAlias alias="Author" type="com.yu000hong.blog.domain.Author"/>
</typeAliases>
{% endhighlight %}

我们在配置别名的时候，可以一次配置一个别名，也可以一次配置整个包。

当我们配置整个包的时候，MyBatis会利用反射将包下面所有的类(排除接口、匿名类和内部类)都进行别名注册。

## @Alias注解

两个问题：

- 我们在注册单个别名时，**alias**属性是可选的，那么最终他的别名是啥呢？
- 我们在注册整个包的时候，包下所有类注册后的别名是啥呢？

这两个问题的答案都一样：取决于类型是否包含`@Alias`注解。

>
> 如果包含`@Alias`注解，那么别名就是由注解指定；如果不包含该注解，那么别名就是**Class.getSimpleName()**。

## ⚠️注意

> 如果我们在注册单个别名时，既使用了**alias**属性，类型又包含了`@Alias`注解，那么使用**alias**属性！

> 别名是不区分大小写的，最后所有别名都是按照小写处理之后进行比较的！

> 如果别名重复定义了，那么MyBatis将会抛出异常！

