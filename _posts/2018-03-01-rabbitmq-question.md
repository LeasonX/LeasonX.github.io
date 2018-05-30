---
layout: post
title: SpirngBoot RabbitMQ 异常:No method found for class [B
date:   2018-03-01
categories: [spring-boot, rabbitmq]
tags: [spring-boot, java]
author: Leason
comment: false
---

## 问题出现

---

在照着纯洁的微笑博客学习rabbitmq的时候，在topic exchange示例时，运行程序总是报 No method found for class [B 异常，具体信息如下:

``` java
org.springframework.amqp.rabbit.listener.exception.ListenerExecutionFailedException: Listener threw exception
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.wrapToListenerExecutionFailedExceptionIfNeeded(AbstractMessageListenerContainer.java:915) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.doInvokeListener(AbstractMessageListenerContainer.java:825) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.invokeListener(AbstractMessageListenerContainer.java:745) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.access$001(SimpleMessageListenerContainer.java:97) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$1.invokeListener(SimpleMessageListenerContainer.java:189) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.invokeListener(SimpleMessageListenerContainer.java:1276) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.executeListener(AbstractMessageListenerContainer.java:726) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.doReceiveAndExecute(SimpleMessageListenerContainer.java:1219) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.receiveAndExecute(SimpleMessageListenerContainer.java:1189) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.access$1500(SimpleMessageListenerContainer.java:97) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1421) [spring-rabbit-1.7.3.RELEASE.jar:na]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_171]
Caused by: org.springframework.amqp.AmqpException: No method found for class [B
	at org.springframework.amqp.rabbit.listener.adapter.DelegatingInvocableHandler.getHandlerForPayload(DelegatingInvocableHandler.java:127) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.adapter.DelegatingInvocableHandler.getMethodNameFor(DelegatingInvocableHandler.java:224) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.adapter.HandlerAdapter.getMethodAsString(HandlerAdapter.java:61) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:140) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.adapter.MessagingMessageListenerAdapter.onMessage(MessagingMessageListenerAdapter.java:106) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.doInvokeListener(AbstractMessageListenerContainer.java:822) ~[spring-rabbit-1.7.3.RELEASE.jar:na]
	... 10 common frames omitted
```

然后网上搜了一下，改了还是报同样的问题。然后对比着教程的示例源码发现了跟我的代码的一些不同: 示例源码的message是String字符串，而我传的是Object对象。于是对着改成String类型的消息，果然好了。

## 思考

---

为什么字符串可以，对象不可以呢，然后我又网上看了message是对象类型的代码示例的时候发现，对象需要序列化，所有我又将对象实体类实现了`Serializable`接口，定义了`serialVersionUID`，再次启动项目。

本以为好了，发现还是报错，异常信息为`Caused by: org.springframework.amqp.AmqpException: No method found for class java.lang.String`，最后发现是原来的队列中存在String类型的消息，最后清空队列中原来的消息，重新启动项目就好啦。清空队列消息可以去rabbitmq的监控页面中`queues`选项，找到purge按钮，点击下就可以啦。

## 总结

---

rabbitmq的message类型为对象的时候需要序列化该实体类，并且最好清空下原来的队列，原来队列中有序列化之前的对象也会报`No method found for class [B`异常。