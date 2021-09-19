---
layout: default
parent: Client
title: Quick tour
nav_order: 1
---
{% include support.md %}

# 简单客户端指南

简单客户端库采用 Enqueue 客户端类和 Symfony 组件，并制作了一个易于使用的客户端门面（facade）。
它减少了开始使用 Enqueue 客户端功能时必须编写的样板代码。

* [安装](#安装)
* [配置](#配置)
* [生产消息](#生产消息)
* [消费消息](#消费消息)

## 安装

```bash
$ composer require enqueue/simple-client enqueue/amqp-ext
```

## 配置

下面的代码展示了如何将简单客户端与 AMQP 传输一起使用。还有其他[受支持的代理](supported_brokers.md)。

```php
<?php
use Enqueue\SimpleClient\SimpleClient;

include __DIR__.'/vendor/autoload.php';

$client = new SimpleClient('amqp:');
```

## 生产消息

客户端可以生成两种类型的消息：事件和命令。
事件用于通知其他人某些事，换句话说，它是[发布订阅模式](https://en.wikipedia.org/wiki/Publish–subscribe_pattern)的实现，有时也称为“即发即忘”。
对于事件，无法获得回复，因为生产者不知道任何已订阅的消费者。
命令用于请求完成一项作业。它是一对一消息传递模式的实现。
生产者可以向消费者请求回复，尽管回复与否取决于消费者。

命令在应用[范围内](message_examples.md#scope)工作，而事件在应用以及[消息总线](message_bus.md)范围内工作。

发送事件示例：

```php
<?php

/** @var \Enqueue\SimpleClient\SimpleClient $client */

$client->setupBroker();

$client->sendEvent('user_updated', 'aMessageData');

// 或一个数组

$client->sendEvent('order_price_calculated', ['foo', 'bar']);

// 或一个JsonSerializable对象
$client->sendEvent('user_activated', new class() implements \JsonSerializable {
    public function jsonSerialize() {
        return ['foo', 'bar'];
    }
});
```

发送命令示例：

```php
<?php

/** @var \Enqueue\SimpleClient\SimpleClient $client */

$client->setupBroker();

// 接受与sendEvent方法相同类型的参数
$client->sendCommand('calculate_statistics', 'aMessageData');

$reply = $client->sendCommand('build_category_tree', 'aMessageData', true);

$replyMessage = $reply->receive(5000); // 等待回复5秒钟

$replyMessage->getBody();
```

## 消费消息

```php
<?php

use Interop\Queue\Message;
use Interop\Queue\Processor;

/** @var \Enqueue\SimpleClient\SimpleClient $client */

$client->bindTopic('a_bar_topic', function(Message $psrMessage) {
    // 处理逻辑

    return Processor::ACK;
});

$client->consume();
```

## Cli命令

```php
#!/usr/bin/env php
<?php

// bin/enqueue.php

use Enqueue\Symfony\Client\SimpleConsumeCommand;
use Enqueue\Symfony\Client\SimpleProduceCommand;
use Enqueue\Symfony\Client\SimpleRoutesCommand;
use Enqueue\Symfony\Client\SimpleSetupBrokerCommand;
use Enqueue\Symfony\Client\SetupBrokerCommand;
use Symfony\Component\Console\Application;

/** @var \Enqueue\SimpleClient\SimpleClient $client */

$application = new Application();
$application->add(new SimpleSetupBrokerCommand($client->getDriver()));
$application->add(new SimpleRoutesCommand($client->getDriver()));
$application->add(new SimpleProduceCommand($client->getProducer()));
$application->add(new SimpleConsumeCommand(
    $client->getQueueConsumer(),
    $client->getDriver(),
    $client->getDelegateProcessor()
));

$application->run();
```

然后运行一下看看那里有什么：

```bash
$ php bin/enqueue.php
```

或消费消息：

```bash
$ php bin/enqueue.php enqueue:consume -vvv --setup-broker
```

[返回目录](../index.md)

