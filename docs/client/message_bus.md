---
layout: default
parent: 客户端
title: 消息总线
nav_order: 4
---
{% include support.md %}

# 消息总线

这是来自[Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html)的关于消息总线的描述

> 消息总线是公共数据模型、公共命令集和消息传递基础设施的组合，允许不同的系统通过一组共享的接口进行通信。

如果您的所有应用都构建在 Enqueue Client 之上，您只需确保它们向共享的主题发送消息。
其余的都是在暗处（hood）下完成的。

如果您想连接另一个应用（例如 Python 应用），您必须遵循以下规则：

* 应用将自己的连接到主题的队列定义为扇出（fanout）。
* 发送到消息总线主题的消息必须有一个 `enqueue.topic_name` 标头。
* 一旦收到消息，它就可以在内部进行路由。`enqueue.topic_name` 标头将会派上用场。

[返回首页](../index.md)