---
layout: default
title: Kafka
parent: Transports
nav_order: 3
---

{% include support.md %}

# Kafka 传输

本传输使用 [Kafka](https://kafka.apache.org/) 流平台作为 MQ 代理。

* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [消费消息](#消费消息)
* [序列化消息](#序列化消息)
* [修改偏移量](#修改偏移量)

## 安装

```bash
$ composer require enqueue/rdkafka
```

## 创建上下文

```php
<?php
use Enqueue\RdKafka\RdKafkaConnectionFactory;

// 连接到localhost:9092
$connectionFactory = new RdKafkaConnectionFactory();

// 同上
$connectionFactory = new RdKafkaConnectionFactory('kafka:');

// 同上
$connectionFactory = new RdKafkaConnectionFactory([]);

// 添加自定义选项，连接到example.com:1000上的Kafka代理
$connectionFactory = new RdKafkaConnectionFactory([
    'global' => [
        'group.id' => uniqid('', true),
        'metadata.broker.list' => 'example.com:1000',
        'enable.auto.commit' => 'false',
    ],
    'topic' => [
        'auto.offset.reset' => 'beginning',
    ],
]);

$context = $connectionFactory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('kafka:')->createContext();
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\RdKafka\RdKafkaContext $context */

$message = $context->createMessage('Hello world!');

$fooTopic = $context->createTopic('foo');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\RdKafka\RdKafkaContext $context */

$message = $context->createMessage('Hello world!');

$fooQueue = $context->createQueue('foo');

$context->createProducer()->send($fooQueue, $message);
```

## 消费消息

```php
<?php
/** @var \Enqueue\RdKafka\RdKafkaContext $context */

$fooQueue = $context->createQueue('foo');

$consumer = $context->createConsumer($fooQueue);

// 启用异步提交以获得更好的性能（自版本0.9.9起默认为true）。
//$consumer->setCommitAsync(true);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
// $consumer->reject($message);
```

## 序列化消息

默认情况下，传输将消息序列化为 json 格式，但您可能希望使用另一种格式，例如[Apache Avro](https://avro.apache.org/docs/1.2.0/)。
为此，您必须实现 `Serializer` 接口并将其设置为上下文、生产者或消费者。
如果序列化器设置为上下文，它将被注入到由上下文创建的所有消费者和生产者。

```php
<?php
use Enqueue\RdKafka\Serializer;
use Enqueue\RdKafka\RdKafkaMessage;

class FooSerializer implements Serializer
{
    public function toMessage($string) {}

    public function toString(RdKafkaMessage $message) {}
}

/** @var \Enqueue\RdKafka\RdKafkaContext $context */

$context->setSerializer(new FooSerializer());
```

## 修改偏移量

默认情况下，消费者从主题的开头开始，并在您处理消息时更新偏移量。
这里有方法可以更改当前的偏移量。

```php
<?php
/** @var \Enqueue\RdKafka\RdKafkaContext $context */

$fooQueue = $context->createQueue('foo');

$consumer = $context->createConsumer($fooQueue);
$consumer->setOffset(123);

$message = $consumer->receive(2000);
```

## 与Symfony bundle一起使用

设置您的 enqueue 以使用 rdkafka 作为您的传输

```yaml
# app/config/config.yml

enqueue:
    default:
        transport: "rdkafka:"
        client: ~
```

如果您不想通过 DSN 字符串来传递它们或需要传递特定选项，您还可以扩展配置以传递其他选项。
由于 rdkafka 使用 librdkafka（基本上是它的封装器），大多数配置选项与 https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md 中的配置选项相同。

```yaml
# app/config/config.yml

enqueue:
    default:
        transport:
            dsn: "rdkafka://"
            global:
                ### 确保这对于每个应用/消费者组都是唯一的，并且不会更改
                ### 否则，Kafka将无法跟踪您的上一个偏移量，并将始终根据 “auto.offset.reset” 设置来启动。
                ### 如果您想了解更多信息，请参阅关于`group.id`属性的Kafka文档
                group.id: 'foo-app'
                metadata.broker.list: 'example.com:1000'
            topic:
                auto.offset.reset: beginning
            ### 自版本0.9.9起，默认情况下异步提交为true。
            ### 建议在早期版本中将其设置为true，否则消费者会变得非常缓慢，
            ### 它会等待偏移量存储在Kafka上，然后再继续。
            commit_async: true
        client: ~
```

[返回目录](index.md)