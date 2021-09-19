---
layout: default
title: Filesystem
parent: Transports
nav_order: 3
---
{% include support.md %}

# Filesystem 传输

使用本地文件系统上的文件作为队列。
它为每个队列\主题创建一个文件。
消息是文件内的一行。
**限制**：它只在自动确认模式下工作，因此如果消费者崩溃，消息就会丢失。本地消息在其他服务器上是不可见的。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [发送限期消息](#发送限期消息)
* [消费消息](#消费消息)
* [清除队列消息](#清除队列消息)

## 安装

```bash
$ composer require enqueue/fs
```

## 创建上下文

```php
<?php
use Enqueue\Fs\FsConnectionFactory;

// 将消息存储在/tmp/enqueue文件夹中
$connectionFactory = new FsConnectionFactory();

// 同上
$connectionFactory = new FsConnectionFactory('file:');

// 存储在自动以文件夹中
$connectionFactory = new FsConnectionFactory('/path/to/queue/dir');

// 同上
$connectionFactory = new FsConnectionFactory('file:///path/to/queue/dir');

// 使用选项
$connectionFactory = new FsConnectionFactory('file:///path/to/queue/dir?pre_fetch_count=1');

// 作为数组
$connectionFactory = new FsConnectionFactory([
    'path' => '/path/to/queue/dir',
    'pre_fetch_count' => 1,
]);

$context = $connectionFactory->createContext();

// 如果已安装enqueue/enqueue库，则可以使用工厂从DSN构建上下文
$context = (new \Enqueue\ConnectionFactoryFactory())->create('file:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Fs\FsContext $context */

$fooTopic = $context->createTopic('aTopic');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Fs\FsContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送限期消息

```php
<?php
/** @var \Enqueue\Fs\FsContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setTimeToLive(60000) // 60秒
    ->send($fooQueue, $message)
;
```

## 消费消息

```php
<?php
/** @var \Enqueue\Fs\FsContext $context */

$fooQueue = $context->createQueue('aQueue');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 清除队列消息：

```php
<?php
/** @var \Enqueue\Fs\FsContext $context */

$fooQueue = $context->createQueue('aQueue');

$context->purge($fooQueue);
```

[返回目录](../index.md)