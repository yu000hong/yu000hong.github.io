---
layout: post
title: 「Spring」@EnableXXX 启用机制
date:   2019-08-13 15:40:00 +0800
tags: [Spring]
---


Spring中我们使用 `@EnableXXX` 系列注解就能轻松启用某项功能，比如：

- @EnableAsync
- @EnableAutoConfiguration
- @EnableScheduling
- @EnableAspectJAutoProxy
- @EnableCaching
- @EnableConfigurationProperties
- @EnableLoadTimeWeaving
- @EnableMBeanExport
- @EnableTransactionManagement

我们观察这些 `@EnableXXX` 的源码可以看出，所有 @EnableXXX 注解都是有 `@Import` 的组合注解，@EnableXXX 的实现其实就是导入了一些自动配置的Bean。@Import 注解的最主要功能就是导入额外的配置信息。

@Import 有三种使用方式：

- @Configuration
- ImportSelector
- ImportBeanDefinitionRegistrar

**官方介绍：**

> Provides functionality equivalent to the `<import/>` element in Spring XML. Allows for importing `@Configuration` classes, `ImportSelector` and `ImportBeanDefinitionRegistrar` implementations, as well as regular component classes.

### 直接导入配置类（导入 @Configuration 类）

{% highlight java %}
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}
{% endhighlight %}

可以看到`@EnableScheduling`注解直接导入配置类`SchedulingConfiguration`，这个类注解了`@Configuration`，且注册了一个`ScheduledAnnotationProcessor`的Bean，SchedulingConfiguration的源码如下：

{% highlight java %}
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class SchedulingConfiguration {

	@Bean(name = TaskManagementConfigUtils.SCHEDULED_ANNOTATION_PROCESSOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public ScheduledAnnotationBeanPostProcessor scheduledAnnotationProcessor() {
		return new ScheduledAnnotationBeanPostProcessor();
	}

}
{% endhighlight %}

### 依据条件选择配置类（实现 ImportSelector 接口）

如果并不确定引入哪个配置类，需要根据`@Import`注解所标识的类或者另一个注解(通常是注解)里的定义信息选择配置类的话，用这种方式。

ImportSelector接口只有一个方法：

{% highlight java %}
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the AnnotationMetadata of the importing @Configuration class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
{% endhighlight %}

> AnnotationMetadata：用来获得当前配置类上的注解。

{% highlight java %}
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {

	Class<? extends Annotation> annotation() default Annotation.class;

	boolean proxyTargetClass() default false;

	AdviceMode mode() default AdviceMode.PROXY;

	int order() default Ordered.LOWEST_PRECEDENCE;

}
{% endhighlight %}

`AsyncConfigurationSelector`继承`AdviceModeImportSelector`，`AdviceModeImportSelector`类实现`ImportSelector`接口 根据**AdviceMode**的不同来选择生成不同的Bean。

{% highlight java %}
public class AsyncConfigurationSelector extends AdviceModeImportSelector<EnableAsync> {

	private static final String ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME =
			"org.springframework.scheduling.aspectj.AspectJAsyncConfiguration";

	@Override
	@Nullable
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {ProxyAsyncConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}

}
{% endhighlight %}

### 动态注册Bean（实现 ImportBeanDefinitionRegistrar 接口）

一般只要用户确切知道哪些Bean需要放入容器的话，自己可以通过 Spring 提供的注解来标识就可以了，比如`@Component`, `@Service`, `@Repository`, `@Bean`等。
如果是不确定的类，或者不是 Spring 专用的，所以并不想用 Spring 的注解进行侵入式标识，那么就可以通过`@Import`注解，实现`ImportBeanDefinitionRegistrar`接口来动态注册Bean。

`ImportBeanDefinitionRegistrar`接口定义：

{% highlight java %}
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing @Configuration class. Note that BeanDefinitionRegistryPostProcessor types may not be
	 * registered here, due to lifecycle constraints related to @Configuration class processing.
	 */
	void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
{% endhighlight %}

比如：

{% highlight java %}
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	boolean proxyTargetClass() default false;
	
	boolean exposeProxy() default false;

}
{% endhighlight %}

`AspectJAutoProxyRegistrar`实现了`ImportBeanDefinitionRegistrar`接口，ImportBeanDefinitionRegistrar的作用是在运行时自动添加Bean到已有的配置类里。

源码：

{% highlight java %}
@Override
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
	AnnotationAttributes enableAspectJAutoProxy =
			AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
	if (enableAspectJAutoProxy != null) {
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}
}
{% endhighlight %}

### 转载

[Spring Boot 自动配置之@Enable* 与@Import注解](https://juejin.im/post/5c761c096fb9a049b41d2299)