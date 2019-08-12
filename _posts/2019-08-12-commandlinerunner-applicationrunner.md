---
layout: post
title: [Spring Boot] CommandLineRunner & ApplicationRunner
date:   2019-08-12 11:30:00 +0800
tags: [Spring Boot]
---

启动成功后，我们可以使用如下几种方式进行初始化操作：

- @PostConstruct 注解
- ApplicationReadyEvent 事件
- CommandLineRunner/ApplicationRunner接口 

这三个方法执行有一定的顺序，依次是：

**@PostConstruct** -> **CommandLineRunner/ApplicationRunner** -> **ApplicationReadyEvent**

{% highlight java %}
@SpringBootApplication
public class DemoApplication implements CommandLineRunner, ApplicationRunner, ApplicationListener<ApplicationReadyEvent> {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("command line runner");
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("application runner");
    }

    @PostConstruct
    public void init(){
        System.out.println("post contruct");
    }

    @Override
    public void onApplicationEvent(ApplicationReadyEvent applicationReadyEvent) {
        System.out.println("ready envent");
    }

}
{% endhighlight %}

上面程序执行之后的输出结果：

```
post contruct
application runner
command line runner
ready envent
```

`CommandLineRunner/ApplicationRunner` 方法可以接收来自命令行的参数，这是和其他两种方式最大的差别！

`CommandLineRunner`接收的所有参数作为一个字符串数组，直接传入到对应的**run**方法里：

{% highlight java %}
@FunctionalInterface
public interface CommandLineRunner {
    void run(String... args) throws Exception;
}
{% endhighlight %}

`ApplicationRunner`会对接收的命令行参数进行解析，所有以`--`开头的参数都会作为Option参数：

{% highlight java %}
@FunctionalInterface
public interface ApplicationRunner {
    void run(ApplicationArguments args) throws Exception;
}

public interface ApplicationArguments {
    String[] getSourceArgs();
    Set<String> getOptionNames();
    boolean containsOption(String name);
    List<String> getOptionValues(String name);
    List<String> getNonOptionArgs();
}
{% endhighlight %}

- `getSourceArgs()`返回所有的参数为一个数组，和 CommandLineRunner 中的参数是一致的

- `getOptionNames()`返回以`--`开头的参数名称

- `getOptionValues(String name)`返回对应参数选项的值

- `getNonOptionArgs()`返回非`--`开头的参数


{% highlight java %}
@SpringBootApplication
public class DemoApplication implements ApplicationRunner {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        for(String s : args.getSourceArgs()){
            System.out.println("source arg: " + s);
        }
        for(String s : args.getOptionNames()){
            System.out.println("option arg: " + s);
        }
        for(String s : args.getNonOptionArgs()){
            System.out.println("non-option arg: " + s);
        }
    }

}
{% endhighlight %}

这段代码执行结果：

```
$ java -jar demo.jar --time=2001 avatar.jpg -t
source arg: --time=2001
source arg: avatar.jpg
source arg: -t
option arg: time
non-option arg: avatar.jpg
non-option arg: -t
```
