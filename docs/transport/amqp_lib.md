---
layout: default
title: AMQP Lib
parent: Transports
nav_order: 3
---
{% include support.md %}

# AMQP 传输

实现 [AMQP 规范](https://www.rabbitmq.com/specification.html) 并实现 [amqp interop](https://github.com/queue-interop/amqp-interop) 接口。
构建在 [php amqp lib](https://github.com/php-amqplib/php-amqplib) 之上。

功能：
* 使用 DSN 字符串配置
* 开箱即用的延迟策略
* 可与其他 AMQP 交互实现互换
* 修复了发送信号时的 AMQPIOWaitException
* 更可靠的心跳实现
* 支持订阅消费者

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
* [长时间运行的任务、心跳、超时](#长时间运行的任务、心跳、超时)

## 安装

```bash
$ composer require enqueue/amqp-lib
```

## 创建上下文

```php
<?php
use Enqueue\AmqpLib\AmqpConnectionFactory;

// 连接到localhost
$factory = new AmqpConnectionFactory();

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
$context = (new \Enqueue\ConnectionFactoryFactory())->create('amqp+lib:')->createContext();
```

## 声明主题

声明主题操作将在代理端创建主题。

```php
<?php
use Interop\Amqp\AmqpTopic;

/** @var \Enqueue\AmqpLib\AmqpContext $context */

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

/** @var \Enqueue\AmqpLib\AmqpContext $context */

$fooQueue = $context->createQueue('foo');
$fooQueue->addFlag(AmqpQueue::FLAG_DURABLE);
$context->declareQueue($fooQueue);

// 要删除队列，请使用删除队列方法。
//$context->deleteQueue($fooQueue);
```

## 绑定队列到主题

将队列连接到主题。因此来自该主题的消息会进入队列并可以被处理。

```php
<?php
use Interop\Amqp\Impl\AmqpBind;

/** @var \Enqueue\AmqpLib\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */
/** @var \Interop\Amqp\Impl\AmqpTopic $fooTopic */

$context->bind(new AmqpBind($fooTopic, $fooQueue));
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\AmqpLib\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpTopic $fooTopic */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\AmqpLib\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送权重消息

```php
<?php
use Interop\Amqp\AmqpQueue;

/** @var \Enqueue\AmqpLib\AmqpContext $context */

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
/** @var \Enqueue\AmqpLib\AmqpContext $context */
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

/** @var \Enqueue\AmqpLib\AmqpContext $context */
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
/** @var \Enqueue\AmqpLib\AmqpContext $context */
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

/** @var \Enqueue\AmqpLib\AmqpContext $context */
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
/** @var \Enqueue\AmqpLib\AmqpContext $context */
/** @var \Interop\Amqp\Impl\AmqpQueue $fooQueue */

$queue = $context->createQueue('aQueue');

$context->purgeQueue($queue);
```

## 长时间运行的任务、心跳、超时

AMQP 依靠心跳功能来确保消费者仍然存在。
基本上，消费者会不时地向 RabbitMQ 代理发送心跳帧，因此代理不会关闭该连接。
由于其同步特性，不可能在 PHP 中实现心跳功能。
您可以在帖子中阅读有关问题的更多信息：[在 PHP 中保持 RabbitMQ 连接](https://blog.mollie.com/keeping-rabbitmq-connections-alive-in-php-b11cb657d5fb)。

`enqueue/amqp-lib`通过将心跳调用注册为[刻度（tick）回调](http://php.net/manual/en/function.register-tick-function.php)来解决该问题。
要使其工作，您必须通过  `declare(ticks=1) {}` 来封装长时间运行的任务。
刻度数可以根据您的需要进行调整。
在每个刻度上调用它并不好。

请注意，如果您将大部分时间花在 IO 操作上，它不会修复心跳问题。

示例：

```php
<?php

use Enqueue\AmqpLib\AmqpConnectionFactory;
use Interop\Amqp\AmqpConsumer;
use Interop\Amqp\AmqpMessage;

$context = (new AmqpConnectionFactory('amqp:?heartbeat_on_tick=1'))->createContext();

$queue = $context->createQueue('a_queue');
$consumer = $context->createConsumer($queue);

$subscriptionConsumer = $context->createSubscriptionConsumer();
$subscriptionConsumer->subscribe($consumer, function(AmqpMessage $message, AmqpConsumer $consumer) {
    // 应调整刻度数
    declare(ticks=1) {
        foreach (fetchHugeSet() as $item) {
            // 长时间周期性做某事，比amqp心跳长得多。
        }
    }

    $consumer->acknowledge($message);

    return true;
});

$subscriptionConsumer->consume(10000);


function fetchHugeSet(): array {};
```

修复部分 `Invalid frame type 65` 问题。

```
Error: Uncaught PhpAmqpLib\Exception\AMQPRuntimeException: Invalid frame type 65 in /some/path/vendor/php-amqplib/php-amqplib/PhpAmqpLib/Connection/AbstractConnection.php:528
```

修复部分 `Broken pipe or closed connection` 问题。

```
PHP Fatal error: Uncaught exception 'PhpAmqpLib\Exception\AMQPRuntimeException' with message 'Broken pipe or closed connection' in /some/path/vendor/php-amqplib/php-amqplib/PhpAmqpLib/Wire/IO/StreamIO.php:190
```

[返回目录](../index.md)