---
layout: default
title: WAMP
parent: 传输
nav_order: 3
---
{% include support.md %}

# Web Application Messaging Protocol (WAMP) 传输

应用于[Web应用消息传递协议](https://wamp-proto.org/)的传输。
WAMP 是一个开放标准的 WebSocket 子协议。
它在内部使用 Thruway 的 [thruway/client](https://github.com/thruway/client) PHP库。

* [安装](#安装)
* [启动WAMP路由](#启动WAMP路由)
* [创建上下文](#创建上下文)
* [消费消息](#消费消息)
* [订阅消费者](#订阅消费者)
* [发送消息到主题](#发送消息到主题)

## 安装

```bash
$ composer require enqueue/wamp
```

## 启动WAMP路由

您可以通过[Thruway](https://github.com/voryx/Thruway)获得 WAMP 路由器：

```bash
$ composer require voryx/thruway
$ php vendor/voryx/thruway/Examples/SimpleWsRouter.php
```

Thruway 现在在 127.0.0.1:9090 上运行


## 创建上下文

```php
<?php
use Enqueue\Wamp\WampConnectionFactory;

$connectionFactory = new WampConnectionFactory();

// 同上
$connectionFactory = new WampConnectionFactory('wamp:');
$connectionFactory = new WampConnectionFactory('ws:');
$connectionFactory = new WampConnectionFactory('wamp://127.0.0.1:9090');

$context = $connectionFactory->createContext();
```

## 消费消息

在向主题发送消息之前启动消息消费者

```php
<?php
/** @var \Enqueue\Wamp\WampContext $context */

$fooTopic = $context->createTopic('foo');

$consumer = $context->createConsumer($fooQueue);

while (true) {
    if ($message = $consumer->receive()) {
        // 处理消息
    }
}
```

## 订阅消费者

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Consumer;

/** @var \Enqueue\Wamp\WampContext $context */
/** @var \Enqueue\Wamp\WampDestination $fooQueue */
/** @var \Enqueue\Wamp\WampDestination $barQueue */

$fooConsumer = $context->createConsumer($fooQueue);
$barConsumer = $context->createConsumer($barQueue);

$subscriptionConsumer = $context->createSubscriptionConsumer();
$subscriptionConsumer->subscribe($fooConsumer, function(Message $message, Consumer $consumer) {
    // 处理消息

    return true;
});
$subscriptionConsumer->subscribe($barConsumer, function(Message $message, Consumer $consumer) {
    // 处理消息

    return true;
});

$subscriptionConsumer->consume(2000); // 2秒
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Wamp\WampContext $context */

$fooTopic = $context->createTopic('foo');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

[返回首页](../index.md)