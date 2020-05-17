---
layout: post
title: 「MyBatis源码分析」基础解析包
date:   2020-05-17 12:00:00 +0800
tags: [MyBatis]
---

MyBatis源码分析系列我们从最基本的配置文件解析开始，今天我们要分析的包：**org.apache.ibatis.parsing**。

这个包里面包含如下文件：

- GenericTokenParser
- PropertyParser
- TokenHandler
- XNode
- XPathParser
- ParsingException

我先说结论吧，我们在配置MyBatis时，可以利用`参数替换`特性，在配置文件中使用特定的参数。MyBatis里面可以使用三种方式：

- `${param}`
- `#{param}`
- `@{param}`

并且，我们可以使用参数默认值(前提是必须启用默认参数值)：

- `${param:defaultValue}`
- `#{param:defaultValue}`
- `@{param:defaultValue}`

两个可控配置属性：

- **org.apache.ibatis.parsing.PropertyParser.enable-default-value**: 是否启用默认参数值，默认值false
- **org.apache.ibatis.parsing.PropertyParser.default-value-separator**: 参数与默认值分隔符，默认值冒号

这两个属性具体在哪里配置，待我们分析了`Configuration`之后再来补充[TODO]。

## TokenHandler

`TokenHandler`是一个接口，它将指定的token参数替换为对应的参数值。

{% highlight java %}
public interface TokenHandler {
  String handleToken(String token);
}
{% endhighlight %}

`TokenHandler`的实现类有：

- VariableTokenHandler
- ParameterMappingTokenHandler
- DynamicCheckerTokenParser
- ParameterMappingTokenHandler
- BindingTokenParser

## PropertyParser

`PropertyParser`是一个工具类，里面含有一个静态类`VariableTokenHandler`和一个静态方法`String parse(String string, Properties variables)`。

我们来看看parse方法：

{% highlight java %}
public static String parse(String string, Properties variables) {
    VariableTokenHandler handler = new VariableTokenHandler(variables);
    GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
    return parser.parse(string);
}
{% endhighlight %}

从代码我们可以看出，`PropertyParser.parse()`方法就是利用`GenericTokenParser`和`VariableTokenHandler`来将字符串中的参数替换出来。

## GenericTokenParser

`GenericTokenParser`的作用是识别出参数占位符，然后利用TokenHandler将它们替换。

我们可以使用`\`对**openToken**和**closeToken**进行转义。

## XPathParser & XNode

`XPathParser`用于XML文件的解析，最终生成`XNode`，用于后续各种**Builder**的构建，如：

- XMLConfigBuilder
- XMLMapperBuilder
