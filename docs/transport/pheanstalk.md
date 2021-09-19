---
layout: default
title: Pheanstalk
parent: 传输
nav_order: 3
---
{% include support.md %}

# Beanstalk (Pheanstalk) 传输

本传输使用 [Beanstalkd](http://kr.github.io/beanstalkd/) 作业管理器。
传输在内部使用 [Pheanstalk](https://github.com/pda/pheanstalk) 库。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [消费消息](#消费消息)

## 安装

```bash
$ composer require enqueue/pheanstalk
```


## 创建上下文

```php
<?php
use Enqueue\Pheanstalk\PheanstalkConnectionFactory;

// 连接到localhost:11300
$factory = new PheanstalkConnectionFactory();

// 同上
$factory = new PheanstalkConnectionFactory('beanstalk:');

// 连接到端口为5555的example主机
$factory = new PheanstalkConnectionFactory('beanstalk://example:5555');

// 同上，但由数组配置
$factory = new PheanstalkConnectionFactory([
    'host' => 'example',
    'port' => 5555
]);

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('beanstalk:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Pheanstalk\PheanstalkContext $context */

$fooTopic = $context->createTopic('aTopic');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Pheanstalk\PheanstalkContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 消费消息

```php
<?php
/** @var \Enqueue\Pheanstalk\PheanstalkContext $context */

$fooQueue = $context->createQueue('aQueue');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive(2000); // 等待2秒

$message = $consumer->receiveNoWait(); // 获取消息或立即返回null

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

[返回首页](../index.md)
