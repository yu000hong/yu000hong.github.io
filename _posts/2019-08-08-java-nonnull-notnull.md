---
layout: post
title: NonNull vs Nonnull vs NotNull
date:   2019-08-08 10:30:00 +0800
tags: [Java]
---

Java 开发中经常会遇到几个常用的注解: **@NonNull**，**@NotNull** 以及 **@Nonnull**。每次用都要查，每次查完又会忘掉，这里特意来总结一下以后用来自查。英文比较全的解释可以参考 [stackoverflow](https://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use) 。

- lombok.NonNull
- javax.annotation.Nonnull
- javax.validation.constraints.NotNull

## lombok.NonNull

这个 annotation 是 [lombok](https://projectlombok.org/features/NonNull) 提供的，根据官方的解释可以看出它是用来辅助生成代码的。如果使用在 **parameter** 前，lombok 将生成一段代码来检测 parameter 是否为 null，如果是则 throw 一个 NullPointerException。 如果使用在 **field** 前，lombok 会在这个 field 相应的生成代码中加入 null 检测。

## javax.annotation.Nonnull

这个很容以和上一次混了，因为名字几乎一样。它实际上只是提供给 IDE 来辅助检测代码中是否非法的传递了 null 给一个 parameter。这是一个静态的检测，所以在运行时不会有任何提示。

## javax.validation.constraints.NotNull

如果项目使用了 Bean 的 validation 框架，那么在定义一些 field 的时候可能会用到 `@NotNull`。这个是用来帮助检测用户输入的，强制某些用户输入值不可以为 null。作用和 `@Max` 和 `@Min` 类似。

## 转载

[Java 里 NonNull 和 NotNull 区别](http://yansu.org/2018/01/03/java-non-null-and-not-null.html)



