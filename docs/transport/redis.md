---
layout: default
title: Redis
parent: 传输
nav_order: 3
---
{% include support.md %}

# Redis 传输

本传输使用[Redis](https://redis.io/)作为消息代理。
它将在那里创建一个集合（队列或主题）。将消息推送到集合的尾部并从头部弹出。
本传输可以运行于 [phpredis](https://github.com/phpredis/phpredis) PHP扩展或[predis](https://github.com/nrk/predis)库。
确保您安装了其中任何一个。

特性：
* 使用 DSN 字符串配置
* 开箱即用的延迟策略
* Recovery&Redelivery support恢复和重新交付支持
* 限期支持
* 延迟支持
* 可与其他队列互操作实现互换
* 支持订阅消费者

目录：
* [安装](#安装)
* [创建上下文](#创建上下文)
* [发送消息到主题](#发送消息到主题)
* [发送消息到队列](#发送消息到队列)
* [发送限期消息](#发送限期消息)
* [发送延迟消息](#发送延迟消息)
* [消费消息](#消费消息)
* [删除队列（清除消息）](#删除队列（清除消息）)
* [删除主题（清除消息）](#删除主题（清除消息）)
* [连接Heroku Redis](#连接Heroku Redis)

## 安装

* 使用 php redis 扩展：

```bash
$ apt-get install php-redis
$ composer require enqueue/redis
```

* 使用 predis 库：

```bash
$ composer require enqueue/redis predis/predis:^1
```

## 创建上下文

* 使用 php redis 扩展：

```php
<?php
use Enqueue\Redis\RedisConnectionFactory;

// 连接到localhost
$factory = new RedisConnectionFactory();

// 同上
$factory = new RedisConnectionFactory('redis:');

// 同上
$factory = new RedisConnectionFactory([]);

// connect to Redis at example.com port 1000 using phpredis extension
$factory = new RedisConnectionFactory([
    'host' => 'example.com',
    'port' => 1000,
    'scheme_extensions' => ['phpredis'],
]);

// 同上，但是使用了DSN字符串。
$factory = new RedisConnectionFactory('redis+phpredis://example.com:1000');

$context = $factory->createContext();

// 如果已安装了 enqueue/enqueue 库，则可以使用工厂从DSN构建上下文。
$context = (new \Enqueue\ConnectionFactoryFactory())->create('redis:')->createContext();

// pass redis instance directly
$redis = new \Enqueue\Redis\PhpRedis([ /** redis connection options */ ]);
$redis->connect();

// Secure\TLS connection. Works only with predis library. Note second "S" in scheme.
$factory = new RedisConnectionFactory('rediss+predis://user:pass@host/0');

$factory = new RedisConnectionFactory($redis);
```

* 使用 predis 库：

```php
<?php
use Enqueue\Redis\RedisConnectionFactory;

$connectionFactory = new RedisConnectionFactory([
    'host' => 'localhost',
    'port' => 6379,
    'scheme_extensions' => ['predis'],
]);

$context = $connectionFactory->createContext();
```

* 使用 predis 和自定义[选项](https://github.com/nrk/predis/wiki/Client-Options):

它可以使您更好地控制供应商的特定功能。

```php
<?php
use Enqueue\Redis\RedisConnectionFactory;
use Enqueue\Redis\PRedis;

$config = [
    'host' => 'localhost',
    'port' => 6379,
    'predis_options' => [
        'prefix'  => 'ns:'
    ]
];

$redis = new PRedis($config);

$factory = new RedisConnectionFactory($redis);
```

## 发送消息到主题

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */

$fooTopic = $context->createTopic('aTopic');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooTopic, $message);
```

## 发送消息到队列

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */

$fooQueue = $context->createQueue('aQueue');
$message = $context->createMessage('Hello world!');

$context->createProducer()->send($fooQueue, $message);
```

## 发送限期消息

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */
/** @var \Enqueue\Redis\RedisDestination $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setTimeToLive(60000) // 60秒
    ->send($fooQueue, $message)
;
```

## 发送延迟消息

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */
/** @var \Enqueue\Redis\RedisDestination $fooQueue */

$message = $context->createMessage('Hello world!');

$context->createProducer()
    ->setDeliveryDelay(5000) // 5秒
    ->send($fooQueue, $message)
;
````

## 消费消息：

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */

$fooQueue = $context->createQueue('aQueue');
$consumer = $context->createConsumer($fooQueue);

$message = $consumer->receive();

// 处理消息

$consumer->acknowledge($message);
//$consumer->reject($message);
```

## 删除队列（清除消息）：

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */

$fooQueue = $context->createQueue('aQueue');

$context->deleteQueue($fooQueue);
```

## 删除主题（清除消息）：

```php
<?php
/** @var \Enqueue\Redis\RedisContext $context */

$fooTopic = $context->createTopic('aTopic');

$context->deleteTopic($fooTopic);
```

## 连接 Heroku Redis

[Heroku Redis](https://devcenter.heroku.com/articles/heroku-redis)描述了如何在 Heroku 上设置 Redis 实例。
要将它与 Enqueue Redis 一起使用，您必须将 REDIS_URL 传递给 RedisConnectionFactory 构造函数。

```php
<?php

// REDIS_URL: redis://h:asdfqwer1234asdf@ec2-111-1-1-1.compute-1.amazonaws.com:111

$connection = new \Enqueue\Redis\RedisConnectionFactory(getenv('REDIS_URL'));
```

[返回首页](../index.md)