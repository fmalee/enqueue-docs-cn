---
layout: default
parent: Magento
title: CLI命令
nav_order: 2
---
{% include support.md %}

# CLI命令

本enqueue Magento 扩展提供了几个命令。
最有用的一个是连接到代理并处理消息的 `enqueue:consume`。
其他命令在调试（如 `enqueue:topics`）或部署（如 `enqueue:setup-broker`）期间可能很有用。

* [enqueue:consume](#enqueueconsume)
* [enqueue:produce](#enqueueproduce)
* [enqueue:setup-broker](#enqueuesetup-broker)
* [enqueue:queues](#enqueuequeues)
* [enqueue:topics](#enqueuetopics)

## enqueue:consume

```
php shell/enqueue.php enqueue:consume --help
用法：
  enqueue:consume [options] [--] [<client-queue-names>]...
  enq:c

参数：
  client-queue-names                     要从中消费消息的队列

选项：
      --message-limit=MESSAGE-LIMIT      消费N条消息并退出
      --time-limit=TIME-LIMIT            限制消费消息的时间
      --memory-limit=MEMORY-LIMIT        消费消息，直到进程达到此内存限制（MB）。
      --setup-broker                     在代理端创建队列、主题、交换、绑定等。
      --idle-timeout=IDLE-TIMEOUT        如果未收到消息时，队列消费者空闲的时间（毫秒）。
      --receive-timeout=RECEIVE-TIMEOUT  队列消费者等待消息的时间（毫秒）
      --skip[=SKIP]                      要跳过的消息消费队列（允许多个值）
  -h, --help                             显示帮助信息
  -q, --quiet                            不要输出任何消息
  -V, --version                          显示应用版本
      --ansi                             强制ANSI输出
      --no-ansi                          禁用ANSI输出
  -n, --no-interaction                   不要问任何互动问题
  -e, --env=ENV                          环境名称。[默认："test"]
      --no-debug                         关闭调试模式
  -v|vv|vvv, --verbose                   增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  处理消息的客户端worker。默认情况下，它连接到默认队列。它根据消息标头选择适当的消息处理器。
```

## enqueue:produce

```
php shell/enqueue.php enqueue:produce --help
用法：
  enqueue:produce <topic> <message>
  enq:p

参数：
  topic                 指定消息发送的主题
  message               指定要发送的消息

选项：
  -h, --help            显示帮助信息
  -q, --quiet           不要输出任何消息
  -V, --version         显示应用版本
      --ansi            强制ANSI输出
      --no-ansi         禁用ANSI输出
  -n, --no-interaction  不要问任何互动问题
  -e, --env=ENV         环境名称。[默认："dev"]
      --no-debug        关闭调试模式
  -v|vv|vvv, --verbose  增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  A command to send a message to topic
```

## enqueue:setup-broker

```
php shell/enqueue.php enqueue:setup-broker --help
用法：
  enqueue:setup-broker
  enq:sb

选项：
  -h, --help            显示帮助信息
  -q, --quiet           不要输出任何消息
  -V, --version         显示应用版本
      --ansi            强制ANSI输出
      --no-ansi         禁用ANSI输出
  -n, --no-interaction  不要问任何互动问题
  -e, --env=ENV         环境名称。[默认："dev"]
      --no-debug        关闭调试模式
  -v|vv|vvv, --verbose  增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  Creates all required queues
```

## enqueue:queues

```
/bin/console enqueue:queues --help
用法：
  enqueue:queues
  enq:m:q
  debug:enqueue:queues

选项：
  -h, --help            显示帮助信息
  -q, --quiet           不要输出任何消息
  -V, --version         显示应用版本
      --ansi            强制ANSI输出
      --no-ansi         禁用ANSI输出
  -n, --no-interaction  不要问任何互动问题
  -e, --env=ENV         环境名称。[默认："dev"]
      --no-debug        关闭调试模式
  -v|vv|vvv, --verbose  增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  命令将显示所有可用的主题以及有关这些主题的一些信息。
```

## enqueue:topics

```
php shell/enqueue.php enqueue:topics --help
用法：
  enqueue:topics
  enq:m:t
  debug:enqueue:topics

选项：
  -h, --help            显示帮助信息
  -q, --quiet           不要输出任何消息
  -V, --version         显示应用版本
      --ansi            强制ANSI输出
      --no-ansi         禁用ANSI输出
  -n, --no-interaction  不要问任何互动问题
  -e, --env=ENV         环境名称。[默认："dev"]
      --no-debug        关闭调试模式
  -v|vv|vvv, --verbose  增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  命令将显示所有可用的主题以及有关这些主题的一些信息。
```

[返回首页](../index.md)

