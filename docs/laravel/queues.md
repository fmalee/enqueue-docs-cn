---
layout: default
parent: Laravel
title: Queues
nav_order: 2
---
{% include support.md %}

# Laravel 队列

[LaravelQueue](https://github.com/php-enqueue/laravel-queue) 包允许以[Laravel方式](https://laravel.com/docs/5.4/queues)来使用于队列交互兼容的传输。
我想你应该已经[安装并配置](quick_tour.md)好了这个包，所以让我们看看你必须做什么才能使队列工作。

## 配置

您必须向 `config/queues.php` 文件添加连接器。驱动必须是 `interop`。

```php
<?php

// config/queue.php

return [
    'default' => 'interop',
    'connections' => [
        'interop' => [
            'driver' => 'interop',
            'dsn' => 'amqp+rabbitmq://guest:guest@localhost:5672/%2f',
        ],
    ],
];
```

这是支持的传输的[完整列表](../transport)。

## 用例

与标准[Laravel 队列](https://laravel.com/docs/5.4/queues)相同。

发送消息示例：

```php
<?php

$job = (new \App\Jobs\EnqueueTest())->onConnection('interop');

dispatch($job);
```

消费消息：

```bash
$ php artisan queue:work interop
```

## Amqp交互

```php
<?php

// config/queue.php

return [
    // 取消注释以将其设置为默认值
    // 'default' => env('QUEUE_DRIVER', 'interop'),

    'connections' => [
        'interop' => [
            'driver' => 'interop',

            // 连接到localhost
            'dsn' => 'amqp:', //

            // 可以是“rabbitmq_dlx”、“rabbitmq_delay_plugin”、DelayStrategy接口实例或null。
            // 'delay_strategy' => 'rabbitmq_dlx'
        ],
    ],
];
```

[返回目录](../index.md)