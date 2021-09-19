---
layout: default
title: STOMP
parent: 传输
nav_order: 3
---
{% include support.md %}

# STOMP 传输

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [消费消息](#消费消息)

## 安装

```bash
$ composer require enqueue/stomp
```

## 创建上下文

```php
<?php
use Enqueue\Stomp\StompConnectionFactory;

// 连接到localhost
$factory = new StompConnectionFactory();

// 同上
$factory = new StompConnectionFactory('stomp:');

// 同上
$factory = new StompConnectionFactory([]);

// 通过stomp连接到RabbitMQ（默认）- 主题名称的前缀为/exchange
$factory = new StompConnectionFactory('stomp+rabbitmq:');

// 通过stomp连接到ActiveMQ - 主题名称的前缀为/topic
$factory = new StompConnectionFactory('stomp+activemq:');

// 连接到端口为100的example.com上的 stomp 代理
$factory = new StompConnectionFactory([
    'host' => 'example.com',
    'port' => 1000,
    'login' => 'theLogin',
]);

// 同上，但是使用了DSN字符串。
$factory = new StompConnectionFactory('stomp://example.com:1000?login=theLogin');

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('stomp:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Stomp\StompContext $context */

$message = $context->createMessage('Hello world!');

$fooTopic = $context->createTopic('foo');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Stomp\StompContext $context */

$message = $context->createMessage('Hello world!');

$fooQueue = $context->createQueue('foo');

$context->createProducer()->send($fooQueue, $message);
```

## 消费消息

```php
<?php
/** @var \Enqueue\Stomp\StompContext $context */

$fooQueue = $context->createQueue('foo');

$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

[返回首页](index.md)
