---
layout: default
nav_exclude: true
---
{% include support.md %}

# Enqueue Monolog 处理器

该包为 [Monolog](https://github.com/Seldaek/monolog) 提供了处理器。
这些处理器允许使用任何  [queue-interop](https://github.com/queue-interop/queue-interop) 兼容的传输将日志发送到 MQ 。

## 安装

您必须先安装 monolog 本身、队列交互处理器和[传输之一](../transport/index.md)。
为简单起见，我们将安装基于 MQ 的文件系统。

```bash
$ composer require enqueue/monolog-queue-handler monolog/monolog enqueue/fs
```

## 用例

```php
<?php

use Monolog\Handler\QueueInteropHandler;
use Monolog\Logger;

require_once __DIR__.'/vendor/autoload.php';

$context = (new \Enqueue\Fs\FsConnectionFactory('file://'.__DIR__.'/queue'))->createContext();

// 创建一个日志频道
$log = new Logger('name');
$log->pushHandler(new QueueInteropHandler($context));

// 添加记录到日志
$log->warning('Foo');
$log->error('Bar');
```

消费者可能看起来像这样：

```php
<?php

use Enqueue\Consumption\QueueConsumer;
use Interop\Queue\Message;
use Interop\Queue\Processor;

require_once __DIR__.'/vendor/autoload.php';

$context = (new \Enqueue\Fs\FsConnectionFactory('file://'.__DIR__.'/queue'))->createContext();

$consumer = new QueueConsumer($context);
$consumer->bindCallback('log', function(Message $message) {
    echo $message->getBody().PHP_EOL;

    return Processor::ACK;
});

$consumer->consume();

```

[返回目录](../index.md)