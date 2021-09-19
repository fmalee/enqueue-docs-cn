---
layout: default
title: MongoDB
parent: 传输
nav_order: 3
---
{% include support.md %}

# Enqueue Mongodb 消息队列传输

允许使用 [MongoDB](https://www.mongodb.com/) 作为消息队列代理。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [发送权重消息](#发送权重消息)
* [发送限期消息](#发送限期消息)
* [发送延迟消息](#发送延迟消息)
* [消费消息](#消费消息)
* [订阅消费者](#订阅消费者)

## 安装

```bash
$ composer require enqueue/mongodb
```

## 创建上下文

```php
<?php
use Enqueue\Mongodb\MongodbConnectionFactory;

// 连接到localhost
$connectionFactory = new MongodbConnectionFactory();

// 同上
$factory = new MongodbConnectionFactory('mongodb:');

// 同上
$factory = new MongodbConnectionFactory([]);

$factory = new MongodbConnectionFactory([
    'dsn' => 'mongodb://localhost:27017/db_name',
    'dbname' => 'enqueue',
    'collection_name' => 'enqueue',
    'polling_interval' => '1000',
]);

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('mongodb:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooTopic */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送权重消息

```php
<?php
/** @var \Enqueue\Mongodb\MongodbContext $context */

$fooQueue = $context->createQueue('foo');

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setPriority(5) // 优先级越高，消息越快到达消费者。
    //
    ->send($fooQueue, $message)
;
```

## 发送限期消息

```php
<?php
/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setTimeToLive(60000) // 60秒
    //
    ->send($fooQueue, $message)
;
```

## 发送延迟消息

```php
<?php
use Enqueue\AmqpTools\RabbitMqDlxDelayStrategy;

/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooQueue */

// 确保运行了 "composer require enqueue/amqp-tools"

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setDeliveryDelay(5000) // 5秒

    ->send($fooQueue, $message)
;
````

## 消费消息

```php
<?php
/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooQueue */

$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 订阅消费者

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Consumer;

/** @var \Enqueue\Mongodb\MongodbContext $context */
/** @var \Enqueue\Mongodb\MongodbDestination $fooQueue */
/** @var \Enqueue\Mongodb\MongodbDestination $barQueue */

$fooConsumer = $context->createConsumer($fooQueue);
$barConsumer = $context->createConsumer($barQueue);

$subscriptionConsumer = $context->createSubscriptionConsumer();
$subscriptionConsumer->subscribe($fooConsumer, function(Message $message, Consumer $consumer) {
    // 处理消息

    $consumer->acknowledge($message);

    return true;
});
$subscriptionConsumer->subscribe($barConsumer, function(Message $message, Consumer $consumer) {
    // 处理消息

    $consumer->acknowledge($message);

    return true;
});

$subscriptionConsumer->consume(2000); // 2秒
```

[返回首页](../index.md)
