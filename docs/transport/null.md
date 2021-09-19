---
layout: default
title: "Null"
parent: Transports
nav_order: 3
---
{% include support.md %}

# NULL 传输

这是一种特殊的传输实现，一种存根。
它不发送也不接收任何东西。
例如在测试中就很有用。

* [安装](#安装)
* [创建上下文](#创建上下文)

## 安装

```bash
$ composer require enqueue/null
```

## 创建上下文

```php
<?php
use Enqueue\Null\NullConnectionFactory;

$connectionFactory = new NullConnectionFactory();

$context = $connectionFactory->createContext();
```

[返回目录](../index.md)