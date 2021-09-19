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
Usage:
  enqueue:consume [options] [--] [<client-queue-names>]...
  enq:c

Arguments:
  client-queue-names                     Queues to consume messages from

Options:
      --message-limit=MESSAGE-LIMIT      Consume n messages and exit
      --time-limit=TIME-LIMIT            Consume messages during this time
      --memory-limit=MEMORY-LIMIT        Consume messages until process reaches this memory limit in MB
      --setup-broker                     Creates queues, topics, exchanges, binding etc on broker side.
      --idle-timeout=IDLE-TIMEOUT        The time in milliseconds queue consumer idle if no message has been received.
      --receive-timeout=RECEIVE-TIMEOUT  The time in milliseconds queue consumer waits for a message.
      --skip[=SKIP]                      Queues to skip consumption of messages from (multiple values allowed)
  -h, --help                             Display this help message
  -q, --quiet                            Do not output any message
  -V, --version                          Display this application version
      --ansi                             Force ANSI output
      --no-ansi                          Disable ANSI output
  -n, --no-interaction                   Do not ask any interactive question
  -e, --env=ENV                          The environment name [default: "test"]
      --no-debug                         Switches off debug mode
  -v|vv|vvv, --verbose                   Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  A client's worker that processes messages. By default it connects to default queue. It select an appropriate message processor based on a message headers
```

## enqueue:produce

```
php shell/enqueue.php enqueue:produce --help
Usage:
  enqueue:produce <topic> <message>
  enq:p

Arguments:
  topic                 A topic to send message to
  message               A message to send

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -e, --env=ENV         The environment name [default: "dev"]
      --no-debug        Switches off debug mode
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  A command to send a message to topic
```

## enqueue:setup-broker

```
php shell/enqueue.php enqueue:setup-broker --help
Usage:
  enqueue:setup-broker
  enq:sb

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -e, --env=ENV         The environment name [default: "dev"]
      --no-debug        Switches off debug mode
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  Creates all required queues
```

## enqueue:queues

```
/bin/console enqueue:queues --help
Usage:
  enqueue:queues
  enq:m:q
  debug:enqueue:queues

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -e, --env=ENV         The environment name [default: "dev"]
      --no-debug        Switches off debug mode
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  A command shows all available queues and some information about them.
```

## enqueue:topics

```
php shell/enqueue.php enqueue:topics --help
Usage:
  enqueue:topics
  enq:m:t
  debug:enqueue:topics

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -e, --env=ENV         The environment name [default: "dev"]
      --no-debug        Switches off debug mode
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Help:
  A command shows all available topics and some information about them.
```

[返回首页](../index.md)

