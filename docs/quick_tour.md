---
layout: default
title: Quick tour
nav_order: 2
---
{% include support.md %}

# 快速指南

* [传输](#传输)
* [消费](#消费)
* [Remote Procedure Call (RPC)](#remote-procedure-call-rpc)
* [客户端](#客户端)
* [Cli命令](#Cli命令)
* [监控](#监控)
* [Symfony bundle](#symfony)

## 传输

传输层或 PSR（Enqueue消息服务）是一个面向消息的中间件，用于在两个或多个客户端之间发送消息。
它是一个消息组件，允许应用创建、发送、接收和读取消息。它允许分布式应用的不同组件之间的通信松散耦合、可靠和异步。

PSR 的灵感来自 JMS（Java消息服务）。我们试图尽可能接近 [JSR 914](https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html) 规范。
目前它支持 [AMQP](https://www.rabbitmq.com/tutorials/amqp-concepts.html) 和 [STOMP](https://stomp.github.io/) 消息队列协议。
您可以连接到许多现代代理，例如 [RabbitMQ](https://www.rabbitmq.com/)、[ActiveMQ](http://activemq.apache.org/) 等。

生产消息：

```php
<?php
use Interop\Queue\ConnectionFactory;

/** @var ConnectionFactory $connectionFactory **/
$context = $connectionFactory->createContext();

$destination = $context->createQueue('foo');
//$destination = $context->createTopic('foo');

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($destination, $message);
```

消费消息:

```php
<?php
use Interop\Queue\ConnectionFactory;

/** @var ConnectionFactory $connectionFactory **/
$context = $connectionFactory->createContext();

$destination = $context->createQueue('foo');
//$destination = $context->createTopic('foo');

$consumer = $context->createConsumer($destination);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 消费

消费是建立在传输功能之上的层。
该组件的目标是简单地消费消息。
`QueueConsumer` 是组件的主要部分，它允许将消息处理器（或回调）绑定到队列。
`consume` 方法则开启一个只要不被中断就持续消费的进程。

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Processor;
use Interop\Queue\Context;
use Enqueue\Consumption\QueueConsumer;

/** @var Context $context */

$queueConsumer = new QueueConsumer($context);

$queueConsumer->bindCallback('foo_queue', function(Message $message) {
    // 处理消息

    return Processor::ACK;
});
$queueConsumer->bindCallback('bar_queue', function(Message $message) {
    // 处理消息

    return Processor::ACK;
});

$queueConsumer->consume();
```

有很多[扩展](consumption/extensions.md)可用。
这里是如何添加它们的示例。
`SignalExtension`提供进程信号的支持，例如，无论何时发送 SIGTERM，它都将得到正确管理。
`LimitConsumptionTimeExtension` 将在给定时间后中断消费。

```php
<?php
use Enqueue\Consumption\ChainExtension;
use Enqueue\Consumption\QueueConsumer;
use Enqueue\Consumption\Extension\SignalExtension;
use Enqueue\Consumption\Extension\LimitConsumptionTimeExtension;

/** @var \Interop\Queue\Context $context */

$queueConsumer = new QueueConsumer($context, new ChainExtension([
    new SignalExtension(),
    new LimitConsumptionTimeExtension(new \DateTime('now + 60 sec')),
]));
```

## Remote Procedure Call (RPC)

这里有一个 RPC 组件可以让您轻松地通过 MQ 发送 RPC 请求。
这是您发送 RPC 消息并等待回复消息的方法。

```php
<?php
use Enqueue\Rpc\RpcClient;

/** @var \Interop\Queue\Context $context */

$queue = $context->createQueue('foo');
$message = $context->createMessage('Hi there!');

$rpcClient = new RpcClient($context);

$promise = $rpcClient->callAsync($queue, $message, 1);
$replyMessage = $promise->receive();
```

消费组件也有扩展。
它简化了 RPC 的服务器端。

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Context;
use Enqueue\Consumption\ChainExtension;
use Enqueue\Consumption\QueueConsumer;
use Enqueue\Consumption\Extension\ReplyExtension;
use Enqueue\Consumption\Result;

/** @var \Interop\Queue\Context $context */

$queueConsumer = new QueueConsumer($context, new ChainExtension([
    new ReplyExtension()
]));

$queueConsumer->bindCallback('foo', function(Message $message, Context $context) {
    $replyMessage = $context->createMessage('Hello');

    return Result::reply($replyMessage);
});

$queueConsumer->consume();
```

## 客户端

它提供了一个易于使用的高级抽象。
该组件的目标是隐藏尽可能多的底层细节，以便您可以专注于真正重要的事情。
例如，它通过创建队列、交换和绑定来为您配置代理。
它提供易于使用的服务来生成和处理消息。
它支持使用统一格式来设置消息过期、延迟、时间戳、关联id。
它支持[消息总线](http://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html)，因此不同的应用可以相互通信。

下面是如何发送和使用**事件消息**的示例：

```php
<?php
use Enqueue\SimpleClient\SimpleClient;
use Interop\Queue\Message;

// composer require enqueue/amqp-ext
$client = new SimpleClient('amqp:');

// composer require enqueue/fs
$client = new SimpleClient('file://foo/bar');
$client->bindTopic('a_foo_topic', function(Message $message) {
    echo $message->getBody().PHP_EOL;

    // 您的事件处理器逻辑
});

$client->setupBroker();

$client->sendEvent('a_foo_topic', 'message');

// 这是一个阻塞调用，它将消费消息直到消息被中断
$client->consume();
```

以及**命令消息**：

```php
<?php
use Enqueue\SimpleClient\SimpleClient;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Enqueue\Client\Config;
use Enqueue\Consumption\Extension\ReplyExtension;
use Enqueue\Consumption\Result;

// composer require enqueue/amqp-ext # or enqueue/amqp-bunny or enqueue/amqp-lib
$client = new SimpleClient('amqp:');

// composer require enqueue/fs
//$client = new SimpleClient('file://foo/bar');

$client->bindCommand('bar_command', function(Message $message) {
    // 您的命令处理器逻辑
});

$client->bindCommand('baz_reply_command', function(Message $message, Context $context) {
    // 您的baz回复命令处理器逻辑

    return Result::reply($context->createMessage('theReplyBody'));
});

$client->setupBroker();

// 它被发送给一个消费者
$client->sendCommand('bar_command', 'aMessageData');

// 有可能得到答复
$promise = $client->sendCommand('bar_command', 'aMessageData', true);

// 只有在开始收到回复后，才能发送多个命令。

$replyMessage = $promise->receive(2000); // 2秒

// 这是一个阻塞调用，它将消费消息直到消息被中断
$client->consume([new ReplyExtension()]);
```

[在此处](client/quick_tour.md#produce-message)阅读有关事件和命令的更多信息。

## Cli命令

该库提供了开箱即用的方便的命令。
它建立在 [Symfony Console 组件](http://symfony.com/doc/current/components/console.html)之上。
最有用的是一个消费命令。其中有两种，一种来自消费组件，另一种来自客户端。

让我们看看如何使用消费一：

```php
#!/usr/bin/env php
<?php
// app.php

use Symfony\Component\Console\Application;
use Interop\Queue\Message;
use Enqueue\Consumption\QueueConsumer;
use Enqueue\Symfony\Consumption\SimpleConsumeCommand;

/** @var QueueConsumer $queueConsumer */

$queueConsumer->bindCallback('a_queue', function(Message $message) {
    // 处理消息
});

$consumeCommand = new SimpleConsumeCommand($queueConsumer);
$consumeCommand->setName('consume');

$app = new Application();
$app->add($consumeCommand);
$app->run();
```

并从控制台开始消费：

```bash
$ app.php consume
```

## 监控

有一个工具可以跟踪已发送/已消费的消息以及消费器性能。[在这里](monitoring.md)阅读更多。

[返回目录](index.md)

## Symfony

[在此处](bundle/quick_tour.md)阅读有关使用 Enqueue 作为 Symfony Bundle 的更多信息。