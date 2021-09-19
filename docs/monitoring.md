---
layout: default
title: Monitoring
nav_order: 95
---

{% include support.md %}

# 监控

Enqueue 提供了一个监控消息队列的工具。
有了它，您可以控制发送的消息数量、成功或失败的处理数量。
有多少消费者正在工作、他们的正常运行时间、处理的消息统计信息、内存使用情况和系统负载。
该工具几乎可以与任何分析和监控平台集成。
有几种集成：

  * [Datadog StatsD](https://datadoghq.com)
  * [InfluxDB](https://www.influxdata.com/) 和 [Grafana](https://grafana.com/)
  * [WAMP (Web Application Messaging Protocol)](https://wamp-proto.org/)
我们正在开发基于 JS\WAMP 的实时 UI 工具，有关更多信息，请[联系我们](mailto:opensource@forma-pro.com)。

![Grafana Monitoring](images/grafana_monitoring.jpg)

如果需要 Grafana 模板，例如图片上的，请[联系我们](mailto:opensource@forma-pro.com)。

* [安装](#安装)
* [追踪发送的消息](#追踪发送的消息)
* [追踪消费的消息](#追踪消费的消息)
* [追踪消费者指标](#追踪消费者指标)
* [消费扩展](#消费扩展)
* [Enqueue队列扩展](#Enqueue队列扩展)
* [InfluxDB存储](#InfluxDB存储)
* [Datadog存储](#Datadog存储)
* [WAMP (Web Socket Messaging Protocol) 存储](#wamp-(web-socket-messaging-protocol)-存储)
* [Symfony应用](#Symfony应用)

## 安装

```bash
$ composer req enqueue/monitoring:0.9.x-dev
```

## 追踪发送的消息

```php
<?php
use Enqueue\Monitoring\SentMessageStats;
use Enqueue\Monitoring\GenericStatsStorageFactory;

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
$statsStorage->pushSentMessageStats(new SentMessageStats(
    (int) (microtime(true) * 1000), // timestamp
    'queue_name', // queue
    'aMessageId',
    'aCorrelationId',
    [], // headers
    []  // properties
));
```

或者，如果您使用 [Queue Interop](https://github.com/queue-interop/queue-interop) 传输，这里是您如何跟踪发送的消息的示例：

```php
<?php
use Interop\Queue\Context;
use Enqueue\Monitoring\SentMessageStats;
use Enqueue\Monitoring\GenericStatsStorageFactory;

/** @var Context $context */

$queue = $context->createQueue('foo');
$message = $context->createMessage('body');

$context->createProducer()->send($queue, $message);

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
$statsStorage->pushSentMessageStats(new SentMessageStats(
    (int) (microtime(true) * 1000),
    $queue->getQueueName(),
    $message->getMessageId(),
    $message->getCorrelationId(),
    $message->getHeaders()[],
    $message->getProperties()
));
```

## 追踪消费的消息

```php
<?php
use Enqueue\Monitoring\ConsumedMessageStats;
use Enqueue\Monitoring\GenericStatsStorageFactory;

$receivedAt = (int) (microtime(true) * 1000);

// heavy processing here.

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
$statsStorage->pushConsumedMessageStats(new ConsumedMessageStats(
    'consumerId',
    (int) (microtime(true) * 1000), // now
    $receivedAt,
    'aQueue',
    'aMessageId',
    'aCorrelationId',
    [], // headers
    [], // properties
    false, // redelivered or not
    ConsumedMessageStats::STATUS_ACK
));
```

或者，如果您使用 [Queue Interop](https://github.com/queue-interop/queue-interop) 传输，这里是您如何跟踪发送的消息的示例：

```php
<?php
use Interop\Queue\Context;
use Enqueue\Monitoring\ConsumedMessageStats;
use Enqueue\Monitoring\GenericStatsStorageFactory;

/** @var Context $context */

$queue = $context->createQueue('foo');

$consumer = $context->createConsumer($queue);

$consumerId = uniqid('consumer-id', true); // we suggest using UUID here
if ($message = $consumer->receiveNoWait()) {
    $receivedAt = (int) (microtime(true) * 1000);

    // heavy processing here.

    $consumer->acknowledge($message);

    $statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
    $statsStorage->pushConsumedMessageStats(new ConsumedMessageStats(
        $consumerId,
        (int) (microtime(true) * 1000), // now
        $receivedAt,
        $queue->getQueueName(),
        $message->getMessageId(),
        $message->getCorrelationId(),
        $message->getHeaders(),
        $message->getProperties(),
        $message->isRedelivered(),
        ConsumedMessageStats::STATUS_ACK
    ));
}
```

## 追踪消费者指标

消费者是长时间运行的进程。了解其中有多少正在运行、它们如何执行、它们使用了多少内存等至关重要。
此示例显示了如何发送此类指标。
在处理消息之间不时的调用此代码。

```php
<?php
use Enqueue\Monitoring\ConsumerStats;
use Enqueue\Monitoring\GenericStatsStorageFactory;

$startedAt = (int) (microtime(true) * 1000);

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
$statsStorage->pushConsumerStats(new ConsumerStats(
    'consumerId',
    (int) (microtime(true) * 1000), // now
    $startedAt,
    null, // finished at
    true, // is started?
    false, // is finished?
    false, // is failed
    ['foo'], // consume from queues
    123, // received messages
    120, // acknowledged messages
    1, // rejected messages
    1, // requeued messages
    memory_get_usage(true),
    sys_getloadavg()[0]
));
```

## 消费扩展

Enqueue [QueueConsumer](quick_tour.md#consumption) 有一个 `ConsumerMonitoringExtension` 扩展。
它可以为您收集消费的消息和消费者统计信息。

```php
<?php
use Enqueue\Consumption\QueueConsumer;
use Enqueue\Consumption\ChainExtension;
use Enqueue\Monitoring\ConsumerMonitoringExtension;
use Enqueue\Monitoring\GenericStatsStorageFactory;
use Interop\Queue\Context;

/** @var Context $context */

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');

$queueConsumer = new QueueConsumer($context, new ChainExtension([
    new ConsumerMonitoringExtension($statsStorage)
]));

// bind

// consume
```

## Enqueue客户端扩展

Enqueue [客户端](quick_tour.md#client) 也有一个 ClientMonitoringExtension 扩展。它可以为您收集已发送的消息统计信息。

## InfluxDB存储

安装额外的软件包：

```bash
$ composer req influxdb/influxdb-php:^1.14
```

```php
<?php
use Enqueue\Monitoring\GenericStatsStorageFactory;

$statsStorage = (new GenericStatsStorageFactory())->create('influxdb://127.0.0.1:8086?db=foo');
```

一些可用的选项：

```
*   'host' => '127.0.0.1',
*   'port' => '8086',
*   'user' => '',
*   'password' => '',
*   'db' => 'enqueue',
*   'measurementSentMessages' => 'sent-messages',
*   'measurementConsumedMessages' => 'consumed-messages',
*   'measurementConsumers' => 'consumers',
*   'client' => null,
*   'retentionPolicy' => null,
```

您可以在 `client` 选项中传递 `InfluxDB\Client` 实例。否则，它将在第一次使用时根据其他选项创建。

如果您的 `InfluxDB\Client` 使用实现了 `InfluxDB\Driver\QueryDriverInterface` 接口路的驱动，那么如果数据库不存在，则会自动为您创建它。默认的 `InfluxDB\Client` 也会这样做。

## Datadog存储

安装额外的软件包：

```bash
$ composer req datadog/php-datadogstatsd:^1.3
```

```php
<?php
use Enqueue\Monitoring\GenericStatsStorageFactory;

$statsStorage = (new GenericStatsStorageFactory())->create('datadog://127.0.0.1:8125');
```

为获得最佳体验，请在公制摘要中调整单位和类型。

示例仪表板：

![Datadog monitoring](images/datadog_monitoring.png)


一些可用的选项（以及所有可用的指标）：

```
*   'host' => '127.0.0.1',
*   'port' => '8125',
*   'batched' => true, // performance boost
*   'global_tags' => '', // should contain keys and values
*   'metric.messages.sent' => 'enqueue.messages.sent',
*   'metric.messages.consumed' => 'enqueue.messages.consumed',
*   'metric.messages.redelivered' => 'enqueue.messages.redelivered',
*   'metric.messages.failed' => 'enqueue.messages.failed',
*   'metric.consumers.started' => 'enqueue.consumers.started',
*   'metric.consumers.finished' => 'enqueue.consumers.finished',
*   'metric.consumers.failed' => 'enqueue.consumers.failed',
*   'metric.consumers.received' => 'enqueue.consumers.received',
*   'metric.consumers.acknowledged' => 'enqueue.consumers.acknowledged',
*   'metric.consumers.rejected' => 'enqueue.consumers.rejected',
*   'metric.consumers.requeued' => 'enqueue.consumers.requeued',
*   'metric.consumers.memoryUsage' => 'enqueue.consumers.memoryUsage',
```


## WAMP (Web Socket Messaging Protocol) 存储

安装额外的软件包：

```bash
$ composer req thruway/pawl-transport:^0.5.0 thruway/client:^0.5.0
```

```php
<?php
use Enqueue\Monitoring\GenericStatsStorageFactory;

$statsStorage = (new GenericStatsStorageFactory())->create('wamp://127.0.0.1:9090?topic=stats');
```

一些可用的选项：

```
*   'host'                => '127.0.0.1',
*   'port'                => '9090',
*   'topic'               => 'stats',
*   'max_retries'         => 15,
*   'initial_retry_delay' => 1.5,
*   'max_retry_delay'     => 300,
*   'retry_delay_growth'  => 1.5,
```

## Symfony应用

您必须注册一些服务才能将监控工具合并到您的 Symfony 应用中。

```yaml
# config/packages/enqueue.yaml

enqueue:
  default:
    transport: 'amqp://guest:guest@bar:5672/%2f'
    monitoring: 'influxdb://127.0.0.1:8086?db=foo'

  another:
    transport: 'amqp://guest:guest@foo:5672/%2f'
    monitoring: 'wamp://127.0.0.1:9090?topic=stats'
    client: ~

  datadog:
    transport: 'amqp://guest:guest@foo:5672/%2f'
    monitoring: 'datadog://127.0.0.1:8125?batched=false'
    client: ~
```

[返回目录](index.md)