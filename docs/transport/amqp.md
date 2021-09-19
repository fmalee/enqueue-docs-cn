---
layout: default
title: AMQP
parent: 传输
nav_order: 3
---
{% include support.md %}

# AMQP 传输

实现 [AMQP 规范](https://www.rabbitmq.com/specification.html) 并实现 [amqp interop](https://github.com/queue-interop/amqp-interop) 接口。
构建在 [php amqp extension](https://github.com/pdezwart/php-amqp) 之上。

缺点：
* [心跳将无法正常工作](https://github.com/pdezwart/php-amqp#persistent-connection)
* [信号不会被正确处理](https://github.com/pdezwart/php-amqp#keeping-track-of-the-workers)

内容：
* [安装](#安装)
* [创建上下文](#创建上下文)
* [声明主题](#声明主题)
* [声明队列](#声明队列)
* [绑定队列到主题](#绑定队列到主题)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [发送权重消息](#发送权重消息)
* [发送限期消息](#发送限期消息)
* [发送延迟消息](#发送延迟消息)
* [消费消息](#消费消息)
* [订阅消费者](#订阅消费者)
* [清除队列消息](#清除队列消息)

## 安装

_**警告**：您需要至少 1.9.3 的 amqp 扩展。以下是从[源代码](https://github.com/php-enqueue/enqueue-dev/blob/09d209447b9dbdf118bff7d983fcb8b0f919e789/docker/Dockerfile#L8)编译扩展的方法。_

```bash
$ composer require enqueue/amqp-ext
```

## 创建上下文

```php
<?php
use Enqueue\AmqpExt\AmqpConnectionFactory;

// 连接到localhost
$connectionFactory = new AmqpConnectionFactory();

// 同上
$factory = new AmqpConnectionFactory('amqp:');

// 同上
$factory = new AmqpConnectionFactory([]);

// 连接到example.com上的AMQP代理
$factory = new AmqpConnectionFactory([
    'host' => 'example.com',
    'port' => 1000,
    'vhost' => '/',
    'user' => 'user',
    'pass' => 'pass',
    'persisted' => false,
]);

// 同上，但是使用了DSN字符串。
$factory = new AmqpConnectionFactory('amqp://user:pass@example.com:10000/%2f');

// // SSL或安全链接
$factory = new AmqpConnectionFactory([
    'dsn' => 'amqps:',
    'ssl_cacert' => '/path/to/cacert.pem',
    'ssl_cert' => '/path/to/cert.pem',
    'ssl_key' => '/path/to/key.pem',
]);

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('amqp:')->createContext();
$context = (new \Enqueue\ConnectionFactoryFactory())->create('amqp+ext:')->createContext();
```

## 声明主题

声明主题操作将在代理端创建主题。

```php
<?php
use Interop\Amqp\AmqpTopic;

/** @var \Enqueue\AmqpExt\AmqpContext $context */

$fooTopic = $context->createTopic('foo');
$fooTopic->setType(AmqpTopic::TYPE_FANOUT);
$context->declareTopic($fooTopic);

// 要删除主题，请使用删除主题方法。
//$context->deleteTopic($fooTopic);
```

## 声明队列

声明队列操作将在代理端创建队列。

```php
<?php
use Interop\Amqp\AmqpQueue;

/** @var \Enqueue\AmqpExt\AmqpContext $context */

$fooQueue = $context->createQueue('foo');
$fooQueue->addFlag(AmqpQueue::FLAG_DURABLE);
$context->declareQueue($fooQueue);

// to remove queue use delete queue method
//$context->deleteQueue($fooQueue);
```

## 绑定队列到主题

将队列连接到主题。因此来自该主题的消息会进入队列并可以被处理。

```php
<?php
use Interop\Amqp\Impl\AmqpBind;

/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */
/** @var \Interop\Amqp\Impl\AmqpTopic $fooTopic */

$context->bind(new AmqpBind($fooTopic, $fooQueue));
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpTopic $fooTopic */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送权重消息

```php
<?php
/** @var \Enqueue\AmqpExt\AmqpContext $context */

$fooQueue = $context->createQueue('foo');
$fooQueue->addFlag(AmqpQueue::FLAG_DURABLE);
$fooQueue->setArguments(['x-max-priority' => 10]);
$context->declareQueue($fooQueue);

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
/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setTimeToLive(60000) // 60秒
    //
    ->send($fooQueue, $message)
;
```

## 发送延迟消息

AMQP 规范没有对于消息延迟的说明，因此生产者抛出 `DeliveryDelayNotSupportedException`。
尽管生产者（和上下文）接受投递延迟策略，并如果设置了，则使用该策略来发送延迟消息。
该 `enqueue/amqp-tools` 包提供了两种 RabbitMQ 延迟策略，要使用它们，您必须安装该软件包。

```php
<?php
use Enqueue\AmqpTools\RabbitMqDlxDelayStrategy;

/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

// 确保运行了 "composer require enqueue/amqp-tools"

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setDelayStrategy(new RabbitMqDlxDelayStrategy())
    ->setDeliveryDelay(5000) // 5秒

    ->send($fooQueue, $message)
;
````

## 消费消息

```php
<?php
/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

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

/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */
/** @var \Interop\Amqp\Impl\AmqpQueue $barQueue */

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

## 清除队列消息

```php
<?php
/** @var \Enqueue\AmqpExt\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

$queue = $context->createQueue('aQueue');

$context->purgeQueue($queue);
```

[返回首页](../index.md)
