---
layout: default
parent: "Symfony bundle"
title: CLI commands
nav_order: 3
---
{% include support.md %}

# Cli命令

EnqueueBundle 提供了几个命令。
最有用的一个 `enqueue:consume` 连接到代理并处理消息。其他命令在调试（如`enqueue:topics`）或部署（如`enqueue:setup-broker`）期间可能很有用。

* [enqueue:consume](#enqueueconsume)
* [enqueue:produce](#enqueueproduce)
* [enqueue:setup-broker](#enqueuesetup-broker)
* [enqueue:routes](#enqueueroutes)
* [enqueue:transport:consume](#enqueuetransportconsume)

## enqueue:consume

```
./bin/console enqueue:consume --help
Usage:
  enqueue:consume [options] [--] [<client-queue-names>]...
  enq:c

Arguments:
  client-queue-names                     要从中消费消息的队列

Options:
      --message-limit=MESSAGE-LIMIT      消费n条消息并退出
      --time-limit=TIME-LIMIT            限制消费消息的时间
      --memory-limit=MEMORY-LIMIT        消费消息，直到进程达到此内存限制（MB）。
      --niceness=NICENESS                设置进程精度
      --setup-broker                     在代理端创建队列、主题、交换、绑定等。
      --receive-timeout=RECEIVE-TIMEOUT  队列消费者等待消息的时间（毫秒）
      --logger[=LOGGER]                  要使用的日志器。可以是"default"、"null"、"stdout"。[default: "default"]
      --skip[=SKIP]                      要跳过的消息消费队列（允许多个值）
  -c, --client[=CLIENT]                  要消费消息的客户端。[default: "default"]
  -h, --help                             显示帮助信息
  -q, --quiet                            不要输出任何消息
  -V, --version                          显示应用版本
      --ansi                             强制ANSI输出
      --no-ansi                          禁用ANSI输出
  -n, --no-interaction                   不要问任何互动问题
  -e, --env=ENV                          环境名称。[default: "test"]
      --no-debug                         关闭调试模式
  -v|vv|vvv, --verbose                   增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

Help:
  处理消息的客户端worker。默认情况下，它连接到默认队列。它根据消息标头选择适当的消息处理器。
```

## enqueue:produce

```
./bin/console enqueue:produce --help
Usage:
  enqueue:produce [options] [--] <message>

Arguments:
  message                  消息

Options:
  -c, --client[=CLIENT]    要向其发送消息的客户端。[default: "default"]
      --topic[=TOPIC]      要向其发送消息的主题
      --command[=COMMAND]  要向其发送消息的命令
  -h, --help               显示帮助信息
  -q, --quiet              不要输出任何消息
  -V, --version            显示应用版本
      --ansi               强制ANSI输出
      --no-ansi            禁用ANSI输出
  -n, --no-interaction     不要问任何互动问题
  -e, --env=ENV            环境名称。[default: "test"]
      --no-debug           关闭调试模式
  -v|vv|vvv, --verbose     增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

Help:
  发送事件到主题
```

## enqueue:setup-broker

```
./bin/console enqueue:setup-broker --help
Usage:
  enqueue:setup-broker [options]
  enq:sb

Options:
  -c, --client[=CLIENT]  要从中消费消息的客户端。 [default: "default"]
  -h, --help             显示帮助信息
  -q, --quiet            不要输出任何消息
  -V, --version          显示应用版本
      --ansi             强制ANSI输出
      --no-ansi          禁用ANSI输出
  -n, --no-interaction   不要问任何互动问题
  -e, --env=ENV          环境名称。[default: "test"]
      --no-debug         关闭调试模式
  -v|vv|vvv, --verbose   增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

Help:
  安装代理。配置代理、创建队列、主题等。
```

## enqueue:routes

```
./bin/console enqueue:routes --help
Usage:
  enqueue:routes [options]
  debug:enqueue:routes

Options:
      --show-route-options  显示隐藏的选项
  -c, --client[=CLIENT]     要从中消费消息的客户端。[default: "default"]
  -h, --help                显示帮助信息
  -q, --quiet               不要输出任何消息
  -V, --version             显示应用版本
      --ansi                强制ANSI输出
      --no-ansi             禁用ANSI输出
  -n, --no-interaction      不要问任何互动问题
  -e, --env=ENV             环境名称。[default: "test"]
      --no-debug            关闭调试模式
  -v|vv|vvv, --verbose      增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

Help:
  命令将列出所有已注册的路由。
```

## enqueue:transport:consume

```
./bin/console enqueue:transport:consume --help
Usage:
  enqueue:transport:consume [options] [--] <processor> [<queues>]...

Arguments:
  processor                              消息处理器
  queues                                 要从中消费的队列

Options:
      --message-limit=MESSAGE-LIMIT      消费n条消息并退出
      --time-limit=TIME-LIMIT            限制消费消息的时间
      --memory-limit=MEMORY-LIMIT        消费消息，直到进程达到此内存限制（MB）。
      --niceness=NICENESS                设置进程精度
      --receive-timeout=RECEIVE-TIMEOUT  队列消费者等待消息的时间（毫秒）
      --logger[=LOGGER]                  要使用的日志器。可以是"default"、"null"、"stdout"。[default: "default"]
  -t, --transport[=TRANSPORT]            要从中使用消息的传输。[default: "default"]
  -h, --help                             显示帮助信息
  -q, --quiet                            不要输出任何消息
  -V, --version                          显示应用版本
      --ansi                             强制ANSI输出
      --no-ansi                          禁用ANSI输出
  -n, --no-interaction                   不要问任何互动问题
  -e, --env=ENV                          环境名称。[default: "test"]
      --no-debug                         关闭调试模式
  -v|vv|vvv, --verbose                   增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

Help:
  消费来自代理的消息的worker。要使用此代理，必须显式设置要从中消费的队列和消息处理器服务
```

[返回目录](index.md)