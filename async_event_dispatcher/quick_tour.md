---
layout: default
nav_exclude: true
---
{% include support.md %}

# 异步事件调度器 (Symfony)

该文档展示了如何在原生 PHP 中设置异步事件调度。
如果您正在寻找在 Symfony 应用中使用它的方法，[请阅读这篇文章](./bundle/async_events.md)

* [安装](#安装)
* [配置](#配置)
* [调度事件](#调度事件)
* [处理异步事件](#处理异步事件)

## 安装

您需要请求异步调度器库和[受支持的传输之一](../transport)

```bash
$ composer require enqueue/async-event-dispatcher enqueue/fs
```

## 配置

```php
<?php

// config.php

use Enqueue\AsyncEventDispatcher\AsyncListener;
use Enqueue\AsyncEventDispatcher\AsyncProcessor;
use Enqueue\AsyncEventDispatcher\PhpSerializerEventTransformer;
use Enqueue\AsyncEventDispatcher\AsyncEventDispatcher;
use Enqueue\AsyncEventDispatcher\SimpleRegistry;
use Enqueue\Fs\FsConnectionFactory;
use Symfony\Component\EventDispatcher\EventDispatcher;

require_once __DIR__.'/vendor/autoload.php';

// 它可以是任何 queue-interop/queue-interop 兼容其它的上下文。
$context = (new FsConnectionFactory('file://'.__DIR__.'/queues'))->createContext();
$eventQueue = $context->createQueue('symfony_events');

$registry = new SimpleRegistry(
    ['the_event' => 'default'],
    ['default' => new PhpSerializerEventTransformer($context)]
);

$asyncListener = new AsyncListener($context, $registry, $eventQueue);

$dispatcher = new EventDispatcher();

// 监听器甚至通过MQ作为消息发送
$dispatcher->addListener('the_event', $asyncListener);

$asyncDispatcher = new AsyncEventDispatcher($dispatcher, $asyncListener);

// 监听器在消费端执行
$asyncDispatcher->addListener('the_event', function() {
});

$asyncProcessor = new AsyncProcessor($registry, $asyncDispatcher);
```

## 调度事件

```php
<?php

// send.php

use Symfony\Component\EventDispatcher\GenericEvent;

require_once __DIR__.'/vendor/autoload.php';

include __DIR__.'/config.php';

$dispatcher->dispatch('the_event', new GenericEvent('theSubject'));
```

## 处理异步事件

```php
<?php

// consume.php

use Interop\Queue\Processor;

require_once __DIR__.'/vendor/autoload.php';
include __DIR__.'/config.php';

$consumer = $context->createConsumer($eventQueue);

while (true) {
    if ($message = $consumer->receive(5000)) {
        $result = $asyncProcessor->process($message, $context);

        switch ((string) $result) {
            case Processor::ACK:
                $consumer->acknowledge($message);
                break;
            case Processor::REJECT:
                $consumer->reject($message);
                break;
            case Processor::REQUEUE:
                $consumer->reject($message, true);
                break;
            default:
                throw new \LogicException('Result is not supported');
        }
    }
}
```

[返回目录](../index.md)