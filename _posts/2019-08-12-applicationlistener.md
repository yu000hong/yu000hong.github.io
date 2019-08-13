---
layout: post
title: 「SpringBoot」SpringApplicationRunListener & ApplicationListener
date:   2019-08-12 11:30:00 +0800
tags: [Spring Boot]
---

## SpringApplicationRunListener

**SpringApplicationRunListener**用于监听 SpringBoot 的整个生命周期，具体定义：

{% highlight java %}
public interface SpringApplicationRunListener {

	/**
	 * Called immediately when the run method has first started. Can be used for very early initialization.
	 */
	void starting();

	/**
	 * Called once the environment has been prepared, but before the ApplicationContext has been created.
	 */
	void environmentPrepared(ConfigurableEnvironment environment);

	/**
	 * Called once the ApplicationContext has been created and prepared, but before sources have been loaded.
	 */
	void contextPrepared(ConfigurableApplicationContext context);

	/**
	 * Called once the application context has been loaded but before it has been refreshed.
	 */
	void contextLoaded(ConfigurableApplicationContext context);

	/**
	 * The context has been refreshed and the application has started but
	 * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
	 * ApplicationRunners} have not been called.
	 */
	void started(ConfigurableApplicationContext context);

	/**
	 * Called immediately before the run method finishes, when the application context has
	 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
	 * {@link ApplicationRunner ApplicationRunners} have been called.
	 */
	void running(ConfigurableApplicationContext context);

	/**
	 * Called when a failure occurs when running the application.
	 */
	void failed(ConfigurableApplicationContext context, Throwable exception);
}
{% endhighlight %}

这些生命周期方法的调用全部都是在`SpringApplication`里面的:

- [starting()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L302)
- [environmentPrepared()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L341)
- [contextPrepared()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L367)
- [contextLoaded()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L386)
- [started()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L318)
- [running()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L327)
- [failed()](https://github.com/spring-projects/spring-boot/blob/faa435459b83b1b47db8bfe4b6bdd0a949aef831/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L793)

我们在程序中可以设置多个`SpringApplicationRunListener`，有个专门的类来处理多个监听器：`SpringApplicationRunListeners`。在程序启动的时候，SpringBoot会从**META-INF/spring.factories**文件中读取所有设置的SpringApplicationRunListener，然后由SpringApplicationRunListeners来统一管理。

> 我们自己定义的SpringApplicationRunListener有一个要求：必须包含两个参数分别为`SpringApplication application`,`String[] args`的构造方法！其中**args**参数就是SpringBoot启动是命令行传入的参数！

## ApplicationListener

**ApplicationListener**也可用于SpringBoot生命周期的监听，但是它更通用，它还可以用于自定义事件的处理！先看定义：

{% highlight java %}
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E event);
}
{% endhighlight %}

```
ApplicationEvent
      |
      |------SpringApplicationEvent
      |             |----ApplicationEnvironmentPreparedEvent
      |             |----ApplicationPreparedEvent
      |             |----ApplicationStartedEvent
      |             |----ApplicationReadyEvent
      |             |----ApplicationFailedEvent
      |             |----ApplicationStartingEvent
      |
      |------ApplicationContextEvent
      |             |----ContextClosedEvent
      |             |----ContextRefreshedEvent
      |             |----ContextStoppedEvent
      |             |----ContextStartedEvent
      |
    .....
    
## 区别与联系

首先说说联系，SpringBoot定义了一个SpringApplicationRunListener的实现类：`EventPublishingRunListener`，这个类的作用就是监听SpringBoot生命周期，并将生命周期变化事件发布到所有的**ApplicationListener**。

源码如下：

{% highlight java %}
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
	private final SpringApplication application;
	private final String[] args;
	private final SimpleApplicationEventMulticaster initialMulticaster;

	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}

	@Override
	public int getOrder() {
		return 0;
	}

	@Override
	public void starting() {
		this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
	}

	@Override
	public void environmentPrepared(ConfigurableEnvironment environment) {
		this.initialMulticaster
				.multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
	}

	@Override
	public void contextPrepared(ConfigurableApplicationContext context) {
		this.initialMulticaster
				.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
	}

	@Override
	public void contextLoaded(ConfigurableApplicationContext context) {
		for (ApplicationListener<?> listener : this.application.getListeners()) {
			if (listener instanceof ApplicationContextAware) {
				((ApplicationContextAware) listener).setApplicationContext(context);
			}
			context.addApplicationListener(listener);
		}
		this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
	}

	@Override
	public void started(ConfigurableApplicationContext context) {
		context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context));
	}

	@Override
	public void running(ConfigurableApplicationContext context) {
		context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
	}

	@Override
	public void failed(ConfigurableApplicationContext context, Throwable exception) {
		ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
		if (context != null && context.isActive()) {
			// Listeners have been registered to the application context so we should
			// use it at this point if we can
			context.publishEvent(event);
		}
		else {
			// An inactive context may not have a multicaster so we use our multicaster to
			// call all of the context's listeners instead
			if (context instanceof AbstractApplicationContext) {
				for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
						.getApplicationListeners()) {
					this.initialMulticaster.addApplicationListener(listener);
				}
			}
			this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
			this.initialMulticaster.multicastEvent(event);
		}
	}

}
{% endhighlight %}

通过源码我们可以看出生命周期的对应关系：

- starting() <-> ApplicationStartingEvent
- environmentPrepared() <-> ApplicationEnvironmentPreparedEvent
- contextPrepared() <-> ApplicationContextInitializedEvent
- contextLoaded() <-> ApplicationPreparedEvent
- started() <-> ApplicationStartedEvent
- running() <-> ApplicationReadyEvent
- failed() <-> ApplicationFailedEvent

再来说下区别，`SpringApplicationRunListener`是通过**spring.factories**方式进行加载的，而`ApplicationListener`可以通过Bean直接被容器加载；其次，`ApplicationListener`可以支持更多自定义事件的监听，不仅仅只有生命周期事件。

## 参考 

[Better application events in Spring Framework 4.2](https://spring.io/blog/2015/02/11/better-application-events-in-spring-framework-4-2)

