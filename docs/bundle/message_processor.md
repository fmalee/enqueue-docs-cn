---
layout: default
parent: "Symfony bundle"
title: 消息处理器
nav_order: 5
---
{% include support.md %}

# 消息处理器

处理器负责处理已消费的消息。
在[消费/消息处理器](../consumption/message_processor.md)中已描述了消息处理器和使用示例，这里我们只展示如何注册一个Enqueue的消息处理器服务。

* 传输:

  * [注册传输处理器](#注册传输处理器)

* 客户端:

  * [注册主题订阅器处理器](#注册主题订阅器处理器)
  * [注册命令订阅器处理器](#注册命令订阅器处理器)
  * [注册自定义处理器](#注册自定义处理器)

## 注册主题订阅器处理器

这里有一个 `TopicSubscriberInterface` 接口（如 [EventSubscriberInterface](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/EventDispatcher/EventSubscriberInterface.php)）。
订阅事件消息很方便。
检查接口描述以获取更多可能的配置方式。
它允许将订阅和处理逻辑保持在一个地方。

```php
<?php
namespace App\Queue;

use Enqueue\Client\TopicSubscriberInterface;
use Interop\Queue\Processor;

class SayHelloProcessor implements Processor, TopicSubscriberInterface
{
    public static function getSubscribedTopics()
    {
        return ['aTopic', 'anotherTopic'];
    }
}
```

容器中用 `enqueue.topic_subscriber` 标签来标记该服务：

```yaml
# config/services.yml

services:
  App\Queue\SayHelloProcessor:
    tags:
        - { name: 'enqueue.topic_subscriber' }

        # 注册到非默认客户端
        - { name: 'enqueue.topic_subscriber', client: 'foo' }
```

## 注册命令订阅器处理器

这里有一个 `CommandSubscriberInterface` 接口。
注册命令处理器很方便。
检查接口描述以获取更多可能的配置方式。
它允许将订阅和处理逻辑保持在一个地方。

```php
<?php
namespace App\Queue;

use Enqueue\Client\CommandSubscriberInterface;
use Interop\Queue\Processor;

class SendEmailProcessor implements Processor, CommandSubscriberInterface
{
    public static function getSubscribedCommand()
    {
        return 'aCommand';
    }
}
```

用 `enqueue.command_subscriber` 标签标记容器中的服务：

```yaml
# config/services.yml

services:
  App\Queue\SendEmailProcessor:
    tags:
        - { name: 'enqueue.command_subscriber' }

        # 注册到非默认客户端
        - { name: 'enqueue.command_subscriber', client: 'foo' }
```

可以注册一个在特定队列（没有其他处理器绑定到它）上工作的命令处理器。
在这种情况下，您可以在不设置任何消息属性的情况下发送消息。
如果您想处理由另一个应用发送的消息，它应该会很方便。

下面是一个配置示例：

```php
<?php
use Enqueue\Client\CommandSubscriberInterface;
use Interop\Queue\Processor;

class SendEmailProcessor implements Processor, CommandSubscriberInterface
{
    public static function getSubscribedCommand()
    {
        return [
           'command' => 'aCommand',
           'queue' => 'the-queue-name',
           'prefix_queue' => false,
           'exclusive' => true,
       ];
    }
}
```

该服务必须使用 `enqueue.command_subscriber`标签进行标记。

# 注册自定义处理器

您可以注册一个既没有实现 `CommandSubscriberInterface` 也没有实现 `TopicSubscriberInterface` 的处理器。
它有一个专用的 `enqueue.processor` 标签，但您必须定义 `topic` 或 `command` 其中一个标签属性。
可以定义您想要向其注册处理器的客户端。默认情况下，它注册到默认客户端（命名为 `default` 的配置或第一个配置）。

```yaml
# src/AppBundle/Resources/services.yml

services:
  AppBundle\Async\SayHelloProcessor:
    tags:
        # 注册为主题处理器
        - { name: 'enqueue.processor', topic: 'aTopic' }
        # 注册为命令处理器
        - { name: 'enqueue.processor', command: 'aCommand' }

        # 注册到非默认客户端
        - { name: 'enqueue.processor', command: 'aCommand', client: 'foo' }
```

该标签有一些额外的选项：

* queue
* prefix_queue
* processor
* exclusive

您可以添加自己的属性。稍后将可以通过 `Route::getOption` 访问它们。

# 注册传输处理器

如果你想拥有一个使用 `enqueue:transport:consume` 的处理器，它应该被标记为 `enqueue.transport.processor`。
可以给您的处理器定义一个指定的传输。默认情况下，它注册到默认传输（命名为 `default` 的配置或第一个配置）。

```yaml
# config/services.yml

services:
  App\Queue\SayHelloProcessor:
    tags:
        - { name: 'enqueue.transport.processor', processor: 'say_hello' }

        # 注册到非默认传输
        - { name: 'enqueue.processor', transport: 'foo' }
```

该标签有一些额外的选项：

* processor

现在您可以运行一个命令并告诉它从给定的队列中消费并使用给定的处理器处理消息：

```bash
$ ./bin/console enqueue:transport:consume say_hello foo_queue -vvv
```

[返回首页](index.md)