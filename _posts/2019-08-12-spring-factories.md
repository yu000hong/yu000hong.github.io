---
layout: post
title: 「Spring」spring.factories & SpringFactoriesLoader
date:   2019-08-12 16:30:00 +0800
tags: [Spring]
---

Spring Framework 内部使用一种工厂加载机制(Factory Loading Mechanism)，也是Spring 提供的SPI(Service Provider Interface)机制。这种机制使用`SpringFactoriesLoader`完成，`SpringFactoriesLoader`使用loadFactories方法加载并实例化从META-INF目录里的**spring.factories**文件出来的工厂，这些**spring.factories**文件都是从classpath里的jar包里找出来的。

## spring.factories

**spring.factories**文件是以Java的Properties格式存在，key是接口、抽象类或注解的全名，value是以逗号 "," 分隔的实现类。比如：

```
example.MyService=example.MyServiceImpl1,example.MyServiceImpl2
```

> **注意**：除了接口和抽象类，key还可以是注解；value是接口和抽象类的实现类，或者具有相应注解的实现类。只要满足：`keyType.isAssignalbeFrom(valueType)`。

## SpringFactoriesLoader

> `SpringFactoriesLoader` loads and instantiates factories of a given type from "META-INF/spring.factories" files which may be present in multiple JAR files in the classpath. The spring.factories file must be in Properties format, where the key is the fully qualified name of the interface or abstract class, and the value is a comma-separated list of implementation class names.

{% highlight java %}
static <T> List<T> loadFactories(Class<T> factoryClass, @Nullable ClassLoader classLoader)
static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader)
{% endhighlight %}

`SpringFactoriesLoader` 会查找并读取所有**META-INF/spring.factories**文件，然后缓存起来避免重复读取。`SpringFactoriesLoader` 能够加载并合并多个**META-INF/spring.factories**文件中同一个KEY下的所有VALUE。如果对应的类实现了`Ordered`接口或者拥有`@Order`注解，那么将按照`Ordered`对应的排序规则(值越小排在越前面)进行排序，没有实现`Ordered`接口的类将给予默认的排序值：**Ordered.LOWEST_PRECEDENCE**。

## ServiceListFactoryBean

Java本身基于`ServiceLoader`提供了SPI机制，Spring除了基于`SpringFactoriesLoader`提供了Spring的SPI机制，同时也集成了Java SPI机制，那就是：`ServiceListFactoryBean`。

**源码**

{% highlight java %}
public class ServiceListFactoryBean extends AbstractServiceLoaderBasedFactoryBean implements BeanClassLoaderAware {
    @Override
    protected Object getObjectToExpose(ServiceLoader<?> serviceLoader) {
        List<Object> result = new LinkedList<>();
        for (Object loaderObject : serviceLoader) {
            result.add(loaderObject);
        }
        return result;
    }
    @Override
    public Class<?> getObjectType() {
        return List.class;
    }
}
{% endhighlight %}

**使用**

{% highlight java %}
@Configuration
public class ServiceConfiguration {
    @Bean
    public ServiceListFactoryBean serviceListFactoryBean() {
        ServiceListFactoryBean serviceListFactoryBean = new ServiceListFactoryBean();
        serviceListFactoryBean.setServiceType(Foo.class);
        return serviceListFactoryBean;
    }
}

List<Foo> list = (List<Foo>)serviceListFactoryBean.getObject();
{% endhighlight %}

## 示例

SpringBoot 里面有很多使用**spring.factories**的例子，如：

{% highlight ini %}
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

# PropertySource Loaders
org.springframework.boot.env.PropertySourceLoader=\
org.springframework.boot.env.PropertiesPropertySourceLoader,\
org.springframework.boot.env.YamlPropertySourceLoader

# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

# FailureAnalysisReporters
org.springframework.boot.diagnostics.FailureAnalysisReporter=\
org.springframework.boot.diagnostics.LoggingFailureAnalysisReporter
{% endhighlight %}

## 参考 

[SpringBoot源码分析之工厂加载机制](https://fangjian0423.github.io/2017/06/05/springboot-factory-loading-mechanism/)

