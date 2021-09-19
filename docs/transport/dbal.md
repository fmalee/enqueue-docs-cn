---
layout: default
title: DBAL
parent: 传输
nav_order: 3
---
{% include support.md %}

# Doctrine DBAL 传输

本传输使用[Doctrine DBAL](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/)库和类似SQL的服务器作为代理。
它在那里创建了一个表。向/从该表推送和弹出消息。

* [安装](#安装)
* [初始化数据库](#初始化数据库)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [发送限期消息](#发送限期消息)
* [发送延迟消息](#发送延迟消息)
* [消费消息](#消费消息)
* [订阅消费者](#订阅消费者)

## 安装

```bash
$ composer require enqueue/dbal
```

## 创建上下文

* 使用配置（内部已创建的连接）：

```php
<?php
use Enqueue\Dbal\DbalConnectionFactory;

$factory = new DbalConnectionFactory('mysql://user:pass@localhost:3306/mqdev');

// 连接到localhost
$factory = new DbalConnectionFactory('mysql:');

$context = $factory->createContext();
```

* 使用现有连接：

```php
<?php
use Enqueue\Dbal\ManagerRegistryConnectionFactory;
use Doctrine\Persistence\ManagerRegistry;

/** @var ManagerRegistry $registry */

$factory = new ManagerRegistryConnectionFactory($registry, [
    'connection_name' => 'default',
]);

$context = $factory->createContext();

// 如果已安装enqueue/enqueue库，则可以使用工厂从DSN构建上下文
$context = (new \Enqueue\ConnectionFactoryFactory())->create('mysql:')->createContext();
```

## 初始化数据库

首先，您必须创建一个表，您的消息将储存在其中。在上下文中有一个方便的`createDataBaseTable`方法。
请注意，必须手动创建数据库。

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $context */

$context->createDataBaseTable();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $context */

$fooTopic = $context->createTopic('aTopic');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送限期消息

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $psrContext */
/** @var \Enqueue\Dbal\DbalDestination $fooQueue */

$message = $psrContext->createMessage('Hello world!');

$psrContext->createProducer()
    ->setTimeToLive(60000) // 60秒
    ->send($fooQueue, $message)
;
```

## 发送延迟消息

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $psrContext */
/** @var \Enqueue\Dbal\DbalDestination $fooQueue */

$message = $psrContext->createMessage('Hello world!');

$psrContext->createProducer()
    ->setDeliveryDelay(5000) // 5秒
    ->send($fooQueue, $message)
;
````

## 消费消息

```php
<?php
/** @var \Enqueue\Dbal\DbalContext $context */

$fooQueue = $context->createQueue('aQueue');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
//$consumer->reject($message);
```

## 订阅消费者

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Consumer;

/** @var \Enqueue\Dbal\DbalContext $context */
/** @var \Enqueue\Dbal\DbalDestination $fooQueue */
/** @var \Enqueue\Dbal\DbalDestination $barQueue */

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