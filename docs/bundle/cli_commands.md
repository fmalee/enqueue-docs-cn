---
layout: default
parent: "Symfony bundle"
title: CLI命令
nav_order: 3
---
{% include support.md %}

# CLI命令

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
用法：
  enqueue:consume [options] [--] [<client-queue-names>]...
  enq:c

参数：
  client-queue-names                     要从中消费消息的队列

选项：
      --message-limit=MESSAGE-LIMIT      消费N条消息并退出
      --time-limit=TIME-LIMIT            限制消费消息的时间
      --memory-limit=MEMORY-LIMIT        消费消息，直到进程达到此内存限制（MB）。
      --niceness=NICENESS                设置进程精度
      --setup-broker                     在代理端创建队列、主题、交换、绑定等。
      --receive-timeout=RECEIVE-TIMEOUT  队列消费者等待消息的时间（毫秒）
      --logger[=LOGGER]                  要使用的日志器。可以是"default"、"null"、"stdout"。[默认："default"]
      --skip[=SKIP]                      要跳过的消息消费队列（允许多个值）
  -c, --client[=CLIENT]                  要消费消息的客户端。[默认："default"]
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
./bin/console enqueue:produce --help
用法：
  enqueue:produce [options] [--] <message>

参数：
  message                  消息

选项：
  -c, --client[=CLIENT]    指定要发送的客户端。[默认："default"]
      --topic[=TOPIC]      指定要发送的主题
      --command[=COMMAND]  指定要发送的命令
  -h, --help               显示帮助信息
  -q, --quiet              不要输出任何消息
  -V, --version            显示应用版本
      --ansi               强制ANSI输出
      --no-ansi            禁用ANSI输出
  -n, --no-interaction     不要问任何互动问题
  -e, --env=ENV            环境名称。[默认："test"]
      --no-debug           关闭调试模式
  -v|vv|vvv, --verbose     增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  发送事件到主题
```

## enqueue:setup-broker

```
./bin/console enqueue:setup-broker --help
用法：
  enqueue:setup-broker [options]
  enq:sb

选项：
  -c, --client[=CLIENT]  要从中消费消息的客户端。 [默认："default"]
  -h, --help             显示帮助信息
  -q, --quiet            不要输出任何消息
  -V, --version          显示应用版本
      --ansi             强制ANSI输出
      --no-ansi          禁用ANSI输出
  -n, --no-interaction   不要问任何互动问题
  -e, --env=ENV          环境名称。[默认："test"]
      --no-debug         关闭调试模式
  -v|vv|vvv, --verbose   增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  安装代理。配置代理、创建队列、主题等。
```

## enqueue:routes

```
./bin/console enqueue:routes --help
用法：
  enqueue:routes [options]
  debug:enqueue:routes

选项：
      --show-route-options  显示隐藏的选项
  -c, --client[=CLIENT]     要从中消费消息的客户端。[默认："default"]
  -h, --help                显示帮助信息
  -q, --quiet               不要输出任何消息
  -V, --version             显示应用版本
      --ansi                强制ANSI输出
      --no-ansi             禁用ANSI输出
  -n, --no-interaction      不要问任何互动问题
  -e, --env=ENV             环境名称。[默认："test"]
      --no-debug            关闭调试模式
  -v|vv|vvv, --verbose      增加消息的详细程度：1表示正常输出，2表示更详细的输出，3表示调试。

帮助：
  命令将列出所有已注册的路由。
```

## enqueue:transport:consume

```
./bin/console enqueue:transport:consume --help
用法：
  enqueue:transport:consume [options] [--] <processor> [<queues>]...

参数：
  processor                              消息处理器
  queues                                 要从中消费的队列

选项：
      --message-limit=MESSAGE-LIMIT      消费N条消息并退出
      --time-limit=TIME-LIMIT            限制消费消息的时间
      --memory-limit=MEMORY-LIMIT        消费消息，直到进程达到此内存限制（MB）。
      --niceness=NICENESS                设置进程精度
      --receive-timeout=RECEIVE-TIMEOUT  队列消费者等待消息的时间（毫秒）
      --logger[=LOGGER]                  要使用的日志器。可以是"default"、"null"、"stdout"。[默认："default"]
  -t, --transport[=TRANSPORT]            要从中使用消息的传输。[默认："default"]
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
  消费来自代理的消息的worker。要使用此代理，必须显式设置要从中消费的队列和消息处理器服务
```

[返回首页](index.md)