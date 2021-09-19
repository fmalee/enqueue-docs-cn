---
layout: default
parent: Client
title: Message examples
nav_order: 2
---
{% include support.md %}

# 消息示例

* [范围](#范围)
* [延迟](#延迟)
* [限期 (TTL)](#限期 (TTL))
* [权重](#权重)
* [时间戳、内容类型、消息 ID](#时间戳、内容类型、消息 ID)

## 范围

有两种类型的可能范围：`Message:SCOPE_MESSAGE_BUS` 和 `Message::SCOPE_APP`。
第一个指示客户端向消息总线发送消息（如果驱动支持），以便其他应用可以使用这些消息。
第二个反过来将消息限制到发送它的应用。没有其他应用可以接收它。

```php
<?php

use Enqueue\Client\Message;

$message = new Message();
$message->setScope(Message::SCOPE_MESSAGE_BUS);

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer->sendEvent('aTopic', $message);
```

## 延迟

设置一个延迟后发送的消息将在延迟时间超过后处理。
一些代理可能不会开箱就支持延迟。
为了在[RabbitMQ](https://www.rabbitmq.com/)中使用延迟功能，您必须安装[延迟插件](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)。

```php
<?php

use Enqueue\Client\Message;

$message = new Message();
$message->setDelay(60); // 秒

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer->sendEvent('aTopic', $message);
```

## 限期 (TTL)

消息可能有过期时间或 TTL（生存时间）。
如果超过了限期时间，但消息仍未被消费，则该消息将从队列中删除。
例如，在最开始的几分钟内发送忘记密码的电邮是有意义的，一个小时后就没有人需要它。

```php
<?php

use Enqueue\Client\Message;

$message = new Message();
$message->setExpire(60); // 秒

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer->sendEvent('aTopic', $message);
```

## 权重

如果您希望某条消息比队列中的其他消息处理得更快，您可以设置优先级。客户端定义了五个优先级常量：

* `MessagePriority::VERY_LOW`
* `MessagePriority::LOW`
* `MessagePriority::NORMAL` （**默认**）
* `MessagePriority::HIGH`
* `MessagePriority::VERY_HIGH`

```php
<?php

use Enqueue\Client\Message;
use Enqueue\Client\MessagePriority;

$message = new Message();
$message->setPriority(MessagePriority::HIGH);

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer->sendEvent('aTopic', $message);
```

## 时间戳、内容类型、消息 ID

这些都是自我描述的东西。
通常它们由客户端设置，因此您不必担心它们。
如果您不喜欢客户端设置的内容，您可以随时设置自定义那些值：

```php
<?php

use Enqueue\Client\Message;

$message = new Message();
$message->setMessageId('aCustomMessageId');
$message->setTimestamp(time());
$message->setContentType('text/plain');

/** @var \Enqueue\Client\ProducerInterface $producer */
$producer->sendEvent('aTopic', $message);
```

[返回目录](../index.md)

