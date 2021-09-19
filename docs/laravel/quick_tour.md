---
layout: default
parent: Laravel
title: 快速指南
nav_order: 1
---
{% include support.md %}

# 快速指南

[enqueue/laravel-queue](https://github.com/php-enqueue/laravel-queue) 是用于Enqueue的消息队列桥接。您可以使用建立在 [queue-interop](https://github.com/queue-interop/queue-interop) 之上的所有传输，包括Enqueue[支持](../transport/index.md)的所有传输。

该包允许您以[laravel的方式](queues.md)来使用队列交互传输，并集成 [Enqueue简单客户端](#Enqueue简单客户端)。

**注意**： 此代码的一部分最初是作为[laravel/framework#20148](https://github.com/laravel/framework/pull/20148)的 PR 提出的。它在没有太多解释的情况下被关闭，所以我决定将它作为一个独立的包开源。

## 安装

您必须安装 `enqueue/laravel-queue` 包和[受支持的传输之一](../transport/index.md)。

```bash
$ composer require enqueue/laravel-queue enqueue/fs
```

## 注册服务提供器

```php
<?php

// config/app.php

return [
    'providers' => [
        Enqueue\LaravelQueue\EnqueueServiceProvider::class,
    ],
];
```

## Laravel队列

在这个阶段，你已经可以使用[laravel 队列](queues.md)了。

## Enqueue简单客户端

如果你想在你的 Laravel 应用中使用 [enqueue/simple-client](https://github.com/php-enqueue/simple-client)，你还需要执行额外的步骤。除了已安装的内容外，您还必须安装客户端库：

```bash
$ composer require enqueue/simple-client
```

创建 `config/enqueue.php` 文件并将客户端配置放在那里。
这是它的示例：

```php
<?php

// config/enqueue.php

return [
    'client' => [
        'transport' => [
            'default' => 'file://'.realpath(__DIR__.'/../storage/enqueue')
        ],
        'client' => [
              'router_topic'             => 'default',
              'router_queue'             => 'default',
              'default_queue'  => 'default',
        ],
    ],
];
```

注册处理器：

```php
<?php
use Enqueue\SimpleClient\SimpleClient;
use Interop\Queue\Message;
use Interop\Queue\Processor;

$app->resolving(SimpleClient::class, function (SimpleClient $client, $app) {
    $client->bindTopic('enqueue_test', function(Message $message) {
        // 在这里做事

        return Processor::ACK;
    });

    return $client;
});

```

发送信息：

```php
<?php
use Enqueue\SimpleClient\SimpleClient;

/** @var SimpleClient $client */
$client = \App::make(SimpleClient::class);

$client->sendEvent('enqueue_test', 'The message');
```

消费消息：

```bash
$ php artisan enqueue:consume -vvv --setup-broker
```

[返回首页](../index.md)