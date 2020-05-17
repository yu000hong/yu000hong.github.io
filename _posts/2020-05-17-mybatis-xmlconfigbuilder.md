---
layout: post
title: 「MyBatis源码分析」MyBatis配置文件解析
date:   2020-05-17 22:13:00 +0800
tags: [MyBatis]
---

MyBatis配置文件大致如下：

{% highlight xml %}
<configuration>
    <properties/>
    <settings/>
    <typeAliases/>
    <typeHandlers/>
    <objectFactory/>
    <objectWrapperFactory/>
    <reflectorFactory/>
    <plugins/>
    <environments/>
    <databaseIdProvider/>
    <mappers/>
</configuration>
{% endhighlight %}

我们使用MyBatis的大致程序框架如下：

- 使用`SqlSessionFactoryBuilder`构建一个`SqlSessionFactory`实例
- 使用`SqlSessionFactory`开启一个会话`SqlSession`(真正的数据库连接在`Transaction`里面)
- 使用`SqlSession`执行mapper对应的SQL
- 处理结果
- 关闭会话

{% highlight java %}
public static void main(String[] args) throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    SqlSession session = sqlSessionFactory.openSession();
    Student stu = session.selectOne("getStudentById", 345229);
    System.out.println(stu.getName());
    session.commit();
    session.close();
}
{% endhighlight %}

`SqlSessionFactoryBuilder`其实是一个工具类，没有实例字段，只有不同形式的`build()`方法。

{% highlight java %}
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
        XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
            reader.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}
{% endhighlight %}

所以，真正的解析工作还是在`XMLConfigBuilder`。我们下面来看看不同节点的解析流程。

## properties

我们可以在配置文件中定义一些属性，也可以引用外部定义的Java属性文件。

{% highlight xml %}
<properties resource="org/mybatis/example/config.properties" url="http://xxx/xxx.properties">
    <property name="username" value="dev_user"/>
    <property name="password" value="F2Fa3!33TYyg"/>
</properties>
{% endhighlight %}

⚠️注意：

- **resource**和**url**最多只能选一个，否则会报错
- 我们最多能引用一个外部Java属性文件
- 所有这些属性会合并，然后被设置为`Configuration.varaiables`字段，用于后续的属性替换

## settings

{% highlight xml %}
<settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="useColumnLabel" value="true"/>
    <setting name="useGeneratedKeys" value="false"/>
</settings>
{% endhighlight %}

这些设置会被反射到`Configuration`对应的setter方法，如：**cacheEnabled**映射到`setCacheEnabled()`方法。如果某个设置通过反射没有找到对应的setter方法，那么将抛出异常。

## typeAliases

{% highlight xml %}
<typeAliases>
    <package name="com.yu000hong.domain"/>
    <typeAlias alias="Author" type="com.yu000hong.blog.domain.Author"/>
</typeAliases>
{% endhighlight %}

类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

MyBatis内置的类型别名有：

_byte => byte | _long => long | _short => short | _int => int | 
_integer => int | _double => double | _float => float | _boolean => boolean |
byte => Byte | long => Long | short => Short | int => Integer |
integer => Integer | double => Double | float => Float | boolean => Boolean |
string => String | date => Date | decimal => BigDecimal | bigdecimal => BigDecimal |
object => Object | map => Map | hashmap => HashMap | list => List |
arraylist => ArrayList | collection => Collection | iterator => Iterator | ... |






