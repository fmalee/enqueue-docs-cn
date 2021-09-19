---
layout: default
title: Amazon SQS
parent: 传输
nav_order: 3
---
{% include support.md %}

# Amazon SQS 传输

[Amazon SQS](https://aws.amazon.com/sqs/)代理的传输。
它在内部使用官方的aws sdk库。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [声明队列](#声明队列)
* [发送消息到队列](#发送消息到队列)
* [发送延迟消息](#发送延迟消息)
* [消费消息](#消费消息)
* [清除队列消息](#清除队列消息)
* [来自另一个 AWS 账户的队列](#来自另一个-AWS-账户的队列)

## 安装

```bash
$ composer require enqueue/sqs
```

## 创建上下文

```php
<?php
use Enqueue\Sqs\SqsConnectionFactory;

$factory = new SqsConnectionFactory([
    'key' => 'aKey',
    'secret' => 'aSecret',
    'region' => 'aRegion',
]);

// 同上，但是使用了DSN字符串。。如果secret包含特殊字符（如+），则可能需要对其进行url编码。
$factory = new SqsConnectionFactory('sqs:?key=aKey&secret=aSecret&region=aRegion');

$context = $factory->createContext();

// 使用预配置的客户端
$client = new Aws\Sqs\SqsClient([ /* ... */ ]);
$factory = new SqsConnectionFactory($client);

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('sqs:')->createContext();
```

## 声明队列

声明队列操作在代理端创建一个队列。

```php
<?php
/** @var \Enqueue\Sqs\SqsContext $context */

$fooQueue = $context->createQueue('foo');
$context->declareQueue($fooQueue);

// 要删除队列，请使用 deleteQueue 方法
//$context->deleteQueue($fooQueue);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Sqs\SqsContext $context */

$fooQueue = $context->createQueue('foo');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送延迟消息

```php
<?php
/** @var \Enqueue\Sqs\SqsContext $context */

$fooQueue = $context->createQueue('foo');
$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setDeliveryDelay(60000) // 60秒

    ->send($fooQueue, $message)
;
```

## 消费消息

```php
<?php
/** @var \Enqueue\Sqs\SqsContext $context */

$fooQueue = $context->createQueue('foo');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 清除队列消息

```php
<?php
/** @var \Enqueue\Sqs\SqsContext $context */

$fooQueue = $context->createQueue('foo');

$context->purgeQueue($fooQueue);
```

## 来自另一个 AWS 账户的队列

SQS 允许使用来自另一个帐户的队列。您可以通过 `queue_owner_aws_account_id` 选项或每个队列使用 `SqsDestination::setQueueOwnerAWSAccountId` 方法来为全局设置所有队列。

```php
<?php
use Enqueue\Sqs\SqsConnectionFactory;

// 全局配置所有队列
$factory = new SqsConnectionFactory('sqs:?queue_owner_aws_account_id=awsAccountId');

$context = (new SqsConnectionFactory('sqs:'))->createContext();

// 每个队列
$queue = $context->createQueue('foo');
$queue->setQueueOwnerAWSAccountId('awsAccountId');
```

## 多区域示例

Enqueue SQS 提供通用的多区域支持。这使用户能够通过在 SqsDestination 上设置区域来指定将命令发送到哪个 AWS 区域。
您可能需要它来访问 SQS FIFO 队列，因为它们不适用于所有区域。
如果未指定，则使用默认区域。

```php
<?php
use Enqueue\Sqs\SqsConnectionFactory;

$context = (new SqsConnectionFactory('sqs:?region=eu-west-2'))->createContext();

$queue = $context->createQueue('foo');
$queue->setRegion('us-west-2');

// 请求发送到US West (Oregon)区域
$context->declareQueue($queue);
```

[返回首页](../index.md)
