---
layout: default
parent: "Symfony bundle"
title: Message producer
nav_order: 4
---
{% include support.md %}

# 消息生产者

您可以直接使用一个传输来发送消息，或是使用客户端。
传输使您可以访问所有特定于传输的功能，因此，您可以在客户端为您提供的易于使用的抽象的地方进行参数调整。

## 传输

```php
<?php

/** @var Symfony\Component\DependencyInjection\ContainerInterface $container */

/** @var Interop\Queue\Context $context */
$context = $container->get('enqueue.transport.[transport_name].context');

$context->createProducer()->send(
    $context->createQueue('a_queue'),
    $context->createMessage('Hello there!')
);
```

## 客户端

客户端附带两种类型的生产者。第一个立即发送消息，另一个（称为假脱机生产者）在内存中收集它们并发送 `onTerminate` 事件（响应已经发送）。

生产者在发送方法上有两种类型：

* `sendEvent` - 消息发送到主题，许多消费者可以订阅它。这是“即发即忘”的策略。事件可以发送到“消息总线”后到达其他应用。
* `sendCommand` - 消息是给一个确切的消费者。它可以用作“即发即忘”或 RPC。命令消息始终在当前应用范围内发送。

### 发送事件

```php
<?php

use Enqueue\Client\ProducerInterface;
use Enqueue\Client\SpoolProducer;

/** @var Symfony\Component\DependencyInjection\ContainerInterface $container */

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer = $container->get(ProducerInterface::class);

// 正在发送消息
$producer->sendEvent('a_topic', 'Hello there!');

/** @var \Enqueue\Client\SpoolProducer $spoolProducer */
$spoolProducer = $container->get(SpoolProducer::class);

// 消息正在 console.terminate 或 kernel.terminate 事件上发送
$spoolProducer->sendEvent('a_topic', 'Hello there!');

// 您可以通过调用flush方法来手动发送排队消息
$spoolProducer->flush();
```

### 发送命令

```php
<?php

use Enqueue\Client\ProducerInterface;
use Enqueue\Client\SpoolProducer;

/** @var Symfony\Component\DependencyInjection\ContainerInterface $container */

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer = $container->get(ProducerInterface::class);

// RPC消息正在发送，我们将其用作RPC
$promise = $producer->sendCommand('a_processor_name', 'Hello there!', $needReply = true);

$replyMessage = $promise->receive();

/** @var \Enqueue\Client\SpoolProducer $spoolProducer */
$spoolProducer = $container->get(SpoolProducer::class);

// 消息正在 console.terminate 或 kernel.terminate 事件上发送
$spoolProducer->sendCommand('a_processor_name', 'Hello there!');

// 您可以通过调用flush方法来手动发送排队消息
$spoolProducer->flush();
```

[返回目录](index.md)