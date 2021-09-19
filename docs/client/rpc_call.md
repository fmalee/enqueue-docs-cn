---
layout: default
parent: 客户端
title: RPC调用
nav_order: 5
---
{% include support.md %}

# RPC调用

客户端的[快速指南](quick_tour.md)介绍了如何获取客户端对象。
这里我们将向您展示如何使用 Enqueue Client 来执行[RPC 调用](https://en.wikipedia.org/wiki/Remote_procedure_call)。
您可以通过定义一个返回某些内容的命令来实现。

## 消费端

在消费端，我们必须注册一个命令处理器，它计算结果并将其发送回发送者。
请注意，您必须添加回复扩展。没有它就行不通。

当然，也可以仅基于传输类来实现 RPC 服务器端。这需要做更多的工作。

```php
<?php

use Interop\Queue\Message;
use Interop\Queue\Context;
use Enqueue\Consumption\Result;
use Enqueue\Consumption\ChainExtension;
use Enqueue\Consumption\Extension\ReplyExtension;
use Enqueue\SimpleClient\SimpleClient;

/** @var \Interop\Queue\Context $context */

// composer require enqueue/amqp-ext # or enqueue/amqp-bunny, enqueue/amqp-lib
$client = new SimpleClient('amqp:');

$client->bindCommand('square', function (Message $message, Context $context) use (&$requestMessage) {
    $number = (int) $message->getBody();

    return Result::reply($context->createMessage($number ^ 2));
});

$client->consume(new ChainExtension([new ReplyExtension()]));
```

## 发送端

在发送端，我们需要一个发送命令并等待回复消息的客户端。

```php
<?php
use Enqueue\SimpleClient\SimpleClient;

$client = new SimpleClient('amqp:');

echo $client->sendCommand('square', 5, true)->receive(5000 /* 5秒 */)->getBody();
```

您可以使用 `sendCommand` 异步执行多个请求，并稍后要求回复。

```php
<?php
use Enqueue\SimpleClient\SimpleClient;

$client = new SimpleClient('amqp:');

/** @var \Enqueue\Rpc\Promise[] $promises */
$promises = [];
$promises[] = $client->sendCommand('square', 5, true);
$promises[] = $client->sendCommand('square', 10, true);
$promises[] = $client->sendCommand('square', 7, true);
$promises[] = $client->sendCommand('square', 12, true);

$replyMessages = [];
while ($promises) {
    foreach ($promises as $index => $promise) {
        if ($replyMessage = $promise->receiveNoWait()) {
            $replyMessages[$index] = $replyMessage;

            unset($promises[$index]);
        }
    }
}
```

[返回首页](../index.md)

