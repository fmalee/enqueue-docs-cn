---
layout: default
title: Amazon SNS-SQS
parent: Transports
nav_order: 3
---
{% include support.md %}

# Amazon SNS-SQS 传输

利用两个 Amazon 的 [SNS-SQS](https://docs.aws.amazon.com/sns/latest/dg/sns-sqs-as-subscriber.html)服务来实现[发布-订阅](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html)企业集成模式。与单个 SQS 传输相反，这增加了将[MessageBus](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html)与队列一起使用的能力。

[Amazon SQS](https://aws.amazon.com/sqs/)代理的传输。
它在内部使用官方[aws sdk 库](https://packagist.org/packages/aws/aws-sdk-php)。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [声明主题、队列并互相绑定](#声明主题、队列并互相绑定)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [消费消息](#消费消息)
* [清除队列消息](#清除队列消息)
* [来自另一个 AWS 账户的队列](#来自另一个-AWS-账户的队列)

## 安装

```bash
$ composer require enqueue/snsqs
```

## 创建上下文

```php
<?php
use Enqueue\SnsQs\SnsQsConnectionFactory;

$factory = new SnsQsConnectionFactory([
    'key' => 'aKey',
    'secret' => 'aSecret',
    'region' => 'aRegion',

    // 或者，您可以使用前缀“sns”、“sqs”来分隔选项
    'key' => 'aKey',              // SNS 和 SQS 的通用选项
    'sns_region' => 'aSnsRegion', // SNS 传输选项
    'sqs_region' => 'aSqsRegion', // SQS 传输选项
]);

// 同上，但是使用了DSN字符串。。如果secret包含特殊字符（如+），则可能需要对其进行url编码。
$factory = new SnsQsConnectionFactory('snsqs:?key=aKey&secret=aSecret&region=aRegion');

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('snsqs:')->createContext();
```

## 声明主题、队列并互相绑定

声明主题、队列操作在代理端创建主题、队列。
绑定会在主题和队列之间创建连接。您将消息发布到主题，主题将消息发送到连接到该主题的每个队列。


```php
<?php
/** @var \Enqueue\SnsQs\SnsQsContext $context */

$inTopic = $context->createTopic('in');
$context->declareTopic($inTopic);

$out1Queue = $context->createQueue('out1');
$context->declareQueue($out1Queue);

$out2Queue = $context->createQueue('out2');
$context->declareQueue($out2Queue);

$context->bind($inTopic, $out1Queue);
$context->bind($inTopic, $out2Queue);

// 要删除主题/队列，请使用deleteTopic/deleteQueue方法
//$context->deleteTopic($inTopic);
//$context->deleteQueue($out1Queue);
//$context->unbind(inTopic, $out1Queue);
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\SnsQs\SnsQsContext $context */

$inTopic = $context->createTopic('in');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($inTopic, $message);
```

## 发送消息到队列

您可以绕过主题并将消息直接发布到队列

```php
<?php
/** @var \Enqueue\SnsQs\SnsQsContext $context */

$fooQueue = $context->createQueue('foo');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```


## 消费消息

```php
<?php
/** @var \Enqueue\SnsQs\SnsQsContext $context */

$out1Queue = $context->createQueue('out1');
$consumer = $context->createConsumer($out1Queue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 清除队列消息

```php
<?php
/** @var \Enqueue\SnsQs\SnsQsContext $context */

$fooQueue = $context->createQueue('foo');

$context->purgeQueue($fooQueue);
```

## 来自另一个 AWS 账户的队列

SNSQS 允许使用来自另一个帐户的队列。您可以通过 `queue_owner_aws_account_id` 选项或每个队列使用 `SnsQsQueue::setQueueOwnerAWSAccountId` 方法来为全局设置所有队列。

```php
<?php
use Enqueue\SnsQs\SnsQsConnectionFactory;

// 全局配置所有队列
$factory = new SnsQsConnectionFactory('snsqs:?sqs_queue_owner_aws_account_id=awsAccountId');

$context = (new SnsQsConnectionFactory('snsqs:'))->createContext();

// 每个队列
$queue = $context->createQueue('foo');
$queue->setQueueOwnerAWSAccountId('awsAccountId');
```

## 多区域示例

Enqueue SNSQS 提供通用的多区域支持。这使用户能够通过在 SnsQsQueue 上设置区域来指定将命令发送到哪个 AWS 区域。
如果未指定，则使用默认区域。

```php
<?php
use Enqueue\SnsQs\SnsQsConnectionFactory;

$context = (new SnsQsConnectionFactory('snsqs:?region=eu-west-2'))->createContext();

$queue = $context->createQueue('foo');
$queue->setRegion('us-west-2');

// 请求发送到US West (Oregon)区域
$context->declareQueue($queue);
```

[返回目录](../index.md)
