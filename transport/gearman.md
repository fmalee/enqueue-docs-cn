---
layout: default
title: Gearman
parent: Transports
nav_order: 3
---
{% include support.md %}

# Gearman 传输

本传输使用 [Gearman](http://gearman.org/) 作业管理器。
本传输在内部使用 [Gearman PHP 扩展](http://php.net/manual/en/book.gearman.php)。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [消费消息](#消费消息)

## 安装

```bash
$ composer require enqueue/gearman
```


## 创建上下文

```php
<?php
use Enqueue\Gearman\GearmanConnectionFactory;

// 连接到localhost:4730
$factory = new GearmanConnectionFactory();

// 同上
$factory = new GearmanConnectionFactory('gearman:');

// 连接到端口为5555的example主机
$factory = new GearmanConnectionFactory('gearman://example:5555');

// 同上，但由数组配置
$factory = new GearmanConnectionFactory([
    'host' => 'example',
    'port' => 5555
]);

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('gearman:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Gearman\GearmanContext $context */

$fooTopic = $context->createTopic('aTopic');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Gearman\GearmanContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 消费消息

```php
<?php
/** @var \Enqueue\Gearman\GearmanContext $context */

$fooQueue = $context->createQueue('aQueue');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive(2000); // 等待2秒

$message = $consumer->receiveNoWait(); // 获取消息或立即返回null

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

[返回目录](../index.md)
