---
layout: default
parent: "Symfony bundle"
title: 异步命令
nav_order: 7
---
{% include support.md %}

# 异步命令

## 安装

```bash
$ composer require enqueue/async-command:0.9.x-dev
```

## 配置

```yaml
# config/packages/enqueue_async_commands.yaml

enqueue:
    default:
        async_commands:
            enabled: true
            timeout: 60
            command_name: ~
            queue_name: ~
```

## 用例

```php
<?php

use Enqueue\Client\ProducerInterface;
use Enqueue\AsyncCommand\Commands;
use Enqueue\AsyncCommand\RunCommand;
use Symfony\Component\DependencyInjection\ContainerInterface;

/** @var $container ContainerInterface */

/** @var ProducerInterface $producer */
$producer = $container->get(ProducerInterface::class);

$cmd = new RunCommand('debug:container', ['--tag=form.type']);
$producer->sendCommand(Commands::RUN_COMMAND, $cmd);
```

您也可以选择获得命令执行的结果：

```php
<?php

use Enqueue\Client\ProducerInterface;
use Enqueue\AsyncCommand\CommandResult;
use Enqueue\AsyncCommand\Commands;
use Enqueue\AsyncCommand\RunCommand;
use Symfony\Component\DependencyInjection\ContainerInterface;

/** @var $container ContainerInterface */

/** @var ProducerInterface $producer */
$producer = $container->get(ProducerInterface::class);

$promise = $producer->sendCommand(Commands::RUN_COMMAND, new RunCommand('debug:container'), true);

// 做其他事情。

if ($replyMessage = $promise->receive(5000)) {
    $result = CommandResult::jsonUnserialize($replyMessage->getBody());

    echo $result->getOutput();
}
```

[返回首页](index.md)