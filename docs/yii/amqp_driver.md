---
layout: default
parent: Yii
title: AMQP Interop 驱动
nav_order: 1
---
{% include support.md %}

# Yii2Queue：AMQP Interop 驱动

_**注意**：这是来自 yiisoft/yii2-queue [仓库](https://github.com/yiisoft/yii2-queue)的 [AMQP Interop 文档](https://github.com/yiisoft/yii2-queue/blob/master/docs/guide/driver-amqp-interop.md) 的副本。_


该驱动与 RabbitMQ 队列一起使用。

为了使其正常工作，您应该将任何与 [amqp interop](https://github.com/queue-interop/queue-interop#amqp-interop) 兼容的传输添加到您的项目中，例如 `enqueue/amqp-lib` 包。

特性：

* 它适用于与 amqp interop 兼容的任何传输，例如

    * 基于 [PHP amqp extension](https://github.com/pdezwart/php-amqp) 的 [enqueue/amqp-ext](https://github.com/php-enqueue/amqp-ext)
    * 基于 [php-amqplib/php-amqplib](https://github.com/php-amqplib/php-amqplib) 的 [enqueue/amqp-lib](https://github.com/php-enqueue/amqp-lib)
    * 基于 [bunny](https://github.com/jakubkulhan/bunny) 的 [enqueue/amqp-bunny](https://github.com/php-enqueue/amqp-bunny)

* 支持权重
* 支持延迟
* 支持 ttr
* 支持尝试
* 包含新选项，例如：vhost、connection_timeout、qos_prefetch_count 等。
* 支持安全 (SSL) AMQP 连接。
* 设置 DSN 的能力，如：amqp:、amqps: 或 amqp://user:pass@localhost:1000/vhost。

配置示例：

```php
return [
    'bootstrap' => [
        'queue', // 该组件注册自己的控制台命令
    ],
    'components' => [
        'queue' => [
            'class' => \yii\queue\amqp_interop\Queue::class,
            'port' => 5672,
            'user' => 'guest',
            'password' => 'guest',
            'queueName' => 'queue',
            'driver' => yii\queue\amqp_interop\Queue::ENQUEUE_AMQP_LIB,

            // 或
            'dsn' => 'amqp://guest:guest@localhost:5672/%2F',

            // 或，同上
            'dsn' => 'amqp:',
        ],
    ],
];
```

控制台
-------

控制台用于监听和处理已队列的任务。

```sh
$ yii queue/listen
```

`listen`命令启动一个守护进程，它无限地查询队列。如果有新任务，它们会立即获取并执行。当命令通过supervisor守护时，此方法最有效。

`listen` 命令的选项：

- `--verbose`、`-v`：将执行状态打印到控制台。
- `--isolate`：作业执行的冗余模式。如果启用，将打印每个作业的执行结果。
- `--color`：高亮冗余模式。

[返回首页](../index.md)
