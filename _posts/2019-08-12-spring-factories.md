---
layout: post
title: 「SpringBoot」spring.factories & SpringFactoriesLoader
date:   2019-08-12 16:30:00 +0800
tags: [SpringBoot]
---

Spring Framework 内部使用一种工厂加载机制(Factory Loading Mechanism)。这种机制使用SpringFactoriesLoader完成，SpringFactoriesLoader使用loadFactories方法加载并实例化从META-INF目录里的spring.factories文件出来的工厂，这些spring.factories文件都是从classpath里的jar包里找出来的。

spring.factories文件是以Java的Properties格式存在，key是接口或抽象类的全名、value是以逗号 "," 分隔的实现类，比如：

```
example.MyService=example.MyServiceImpl1,example.MyServiceImpl2
```

> SpringFactoriesLoader loads and instantiates factories of a given type from "META-INF/spring.factories" files which may be present in multiple JAR files in the classpath. The spring.factories file must be in Properties format, where the key is the fully qualified name of the interface or abstract class, and the value is a comma-separated list of implementation class names.

{% highlight java %}
static <T> List<T> loadFactories(Class<T> factoryClass, @Nullable ClassLoader classLoader)
static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader)
{% endhighlight %}

`SpringFactoriesLoader` 会查找并读取所有**META-INF/spring.factories**文件，然后缓存起来避免重复读取。

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

