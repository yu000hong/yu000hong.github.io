---
layout: post
title: 各消息队列产品比较
date: 2019-01-29 17:35:00 +0800
tags: [消息队列]
---

一些主流的消息队列：
- Redis
- Kafka
- RabbitMQ
- ActiveMQ
- RocketMQ
- ZeroMQ

## Redis

Redis作为消息队列使用时，多用于实时性较高的消息推送，可以有两种方式：
- 基于Pub/Sub
- 基于List

Redis的消息队列消费模式里并没有分组的功能，基于Pub/Sub方式的消息队列是所有订阅者都能收到并消费消息，而基于List的消息队列的订阅者是共同消费消息（A消费了，B就不能消费该消息了）。Kafka的消费模式就比较灵活，同时支持这两种形式。

**基于Pub/Sub**

基于Pub/Sub的消息队列，并不保证可靠，而且断电就清空。

**基于List**

使用List作为消息队列虽然有持久化，但是又太弱智，也并非完全可靠不会丢。

## 参考资料

[Understanding When to use RabbitMQ or Apache Kafka](https://content.pivotal.io/blog/understanding-when-to-use-rabbitmq-or-apache-kafka)
