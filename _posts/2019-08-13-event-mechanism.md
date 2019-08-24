---
layout: post
title: 「Spring」事件机制
date:   2019-08-13 10:11:00 +0800
tags: [Spring]
---

Spring的事件机制是一套相当灵活的机制，使用它可以简便地将我们的代码解耦从而优化我们的代码。举个例子，假设有一个添加评论的方法，在评论添加成功之后需要进行修改redis缓存、给用户添加积分等等操作。当然可以在添加评论的代码后面假设这些操作，但是这样的代码违反了设计模式的多项原则：单一职责原则、迪米特法则、开闭原则。一句话说就是耦合性太大了，比如将来评论添加成功之后还需要有另外一个操作，这时候我们就需要去修改我们的添加评论代码了。

在以前的代码中，我使用观察者模式来解决这个问题。不过Spring中已经存在了一个升级版观察者模式的机制，这就是监听者模式。通过该机制我们就可以发送接收任意的事件并处理。

Spring事件机制涉及到的包为：`org.springframework.context.event`，涉及到的类主要有：

- ApplicationEvent：事件接口
- ApplicationListener：事件监听器，负责监听各类事件
- ApplicationEventPublisher：事件发布器，它的实现者都是`ApplicationContext`，我们只要获取到ApplicationContext就可以发布任何事件
- ApplicationEventMulticaster：事件分发器，注册管理各类监听器以及分发由ApplicationContext发布的事件
- EventListener：事件监听器注解，我们可以不用实现`ApplicationListener`接口，直接使用该注解到达相同的目的

## 简单示例

通过一个简单的demo来看看Spring事件通知的使用：

{% highlight java %}
// 定义一个事件
public class EventDemo extends ApplicationEvent {
    private String message;

    public EventDemo(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

// 定义一个事件监听者
@Component
public class EventDemoListener implements ApplicationListener<EventDemo> {
    @Override
    public void onApplicationEvent(EventDemo event) {
        System.out.println("receiver " + event.getMessage());
    }
}

// 事件发布
@Component
public class EventDemoPublish {
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void publish(String message) {
        EventDemo demo = new EventDemo(this, message);
        applicationEventPublisher.publishEvent(demo);
    }
}
{% endhighlight %}





## 参考

[Spring Event事件通知机制](https://blog.wangqi.love/articles/Java/Spring%20Event%E4%BA%8B%E4%BB%B6%E9%80%9A%E7%9F%A5%E6%9C%BA%E5%88%B6.html)