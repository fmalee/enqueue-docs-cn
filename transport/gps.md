---
layout: default
title: GPS
parent: Transports
nav_order: 3
---
{% include support.md %}

# Google Pub Sub 传输

[Google Pub Sub](https://cloud.google.com/pubsub/docs/) 云 MQ 的传输。
它使用内部 google sdk 官方库 [google/cloud-pubsub](https://packagist.org/packages/google/cloud-pubsub)。

- [安装](https://php-enqueue.github.io/transport/gps/#installation)

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [消费消息](#消费消息)

## 安装

```bash
$ composer require enqueue/gps
```

## 创建上下文

要启用 Google Cloud Pub/Sub Emulator，请设置 `PUBSUB_EMULATOR_HOST` 环境变量。这里有一个方便的 docker 容器 [google/cloud-sdk](https://hub.docker.com/r/google/cloud-sdk/)。

```php
<?php
use Enqueue\Gps\GpsConnectionFactory;

putenv('PUBSUB_EMULATOR_HOST=http://localhost:8900');

$connectionFactory = new GpsConnectionFactory();

// 同上
$connectionFactory = new GpsConnectionFactory('gps:');

$context = $connectionFactory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('gps:')->createContext();
```

## 发送消息到主题

在发送消息之前，您必须声明一个主题。
该操作在代理端创建一个主题。
Google 只允许将消息发送到主题。

```php
<?php
/** @var \Enqueue\Gps\GpsContext $context */

$fooTopic = $context->createTopic('foo');
$message = $context->createMessage('Hello world!');

$context->declareTopic($fooTopic);

$context->createProducer()->send($fooTopic, $message);
```

## 消费消息

在您使用消息之前，您必须将一个队列订阅到主题。
Google 不允许直接使用来自主题的消息。

```php
<?php
/** @var \Enqueue\Gps\GpsContext $context */

$fooTopic = $context->createTopic('foo');
$fooQueue = $context->createQueue('foo');

$context->subscribe($fooTopic, $fooQueue);

$consumer = $context->createConsumer($fooQueue);
$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

[返回目录](../index.md)
