---
layout: default
parent: "Symfony bundle"
title: 快速指南
nav_order: 1
---
{% include support.md %}

# 快速指南

本 [EnqueueBundle](https://github.com/php-enqueue/enqueue-bundle) 集成了队列库。
它添加了易于使用的[配置层](config_reference.md)、注册服务以及方便的[cli 命令](cli_commands.md)。

## 安装

```bash
$ composer require enqueue/enqueue-bundle enqueue/fs
```

_**注意**: 您可以使用其他 [传输](../transport/index.md)。_

_**注意**: 如果您正在寻找一种从 `php-amqplib/rabbitmq-bundle` 迁移的方式，请阅读[本文](https://blog.forma-pro.com/the-how-and-why-of-the-migration-from-rabbitmqbundle-to-enqueuebundle-6c4054135e2b)。_

## 启用Bundle

然后，将  `new Enqueue\Bundle\EnqueueBundle()` 添加到您项目的 `app/AppKernel.php` 文件的 `registerBundles` 方法中的bundle数组中：

```php
<?php
// src/Kernel.php
namespace App;

use Symfony\Component\HttpKernel\Kernel as BaseKernel;

class Kernel extends BaseKernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...

            new Enqueue\Bundle\EnqueueBundle(),
        );

        // ...
    }

    // ...
}
```

## 用例

首先，您必须配置一个传输层。
如果需要，您可以选择配置多个传输。
基于以下内容，其中一个将自动成为默认传输：

1. 如果有一个名为 `default` 的传输，那么它将成为默认值。
2. 未特别指定时的第一个传输。

默认传输的服务将在它们各自的类接口下的常用 Symfony 容器中提供给您（见下文）

```yaml
# app/config/config.yml

enqueue:
    default:
        transport: "amqp:"
        client: ~
    some_other_transport:
        transport: "amqp:"
        client: ~
```

配置完所有内容后，您就可以开始生产消息了。
如前所述，默认传输服务已在容器中可用。在这里，我们使用 `ProducerInterface` 向 `default` 传输生产消息。

```php
<?php
use Enqueue\Client\Message;
use Enqueue\Client\ProducerInterface;

/** @var ProducerInterface $producer **/
$producer = $container->get(ProducerInterface::class);

// 如果您想要一个有别于默认的生产者（例如上面示例中指定的另一个生产者），则使用
// $producer = $container->get('enqueue.client.some_other_transport.producer');

// 给若干消费者发送事件
$producer->sendEvent('aFooTopic', 'Something has happened');
// 如果需要更大的灵活性，还可以给第二个参数传递一个Enqueue\Client\Message的实例。
$properties = [];
$headers = [];
$message = new Message('Message body', $properties, $headers);
$producer->sendEvent('aBarTopic', $message);

// 给一个消费者发送命令
$producer->sendCommand('aProcessorName', 'Something has happened');
```

要消费消息，您必须首先创建一个消息处理器。

下面的示例展示了如何创建一个处理器来接收来自 `aFooTopic` 主题（并且只有这个主题）的消息。
它假定您使用默认的 Symfony 服务配置，并且此类已[自动装配](https://symfony.com/doc/current/service_container.html#the-autoconfigure-option)。否则，您将不得不手动标记它。
如果您使用多种传输，则尤其如此：如果保持自动装配，处理器将仅连接到默认传输。

注意：Enqueue的主题和某些传输（例如 Kafka）上的主题是两件不同的事情。

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Context;
use Interop\Queue\Processor;
use Enqueue\Client\TopicSubscriberInterface;

class FooProcessor implements Processor, TopicSubscriberInterface
{
    public function process(Message $message, Context $session)
    {
        echo $message->getBody();

        return self::ACK;
        // return self::REJECT; // 当消息已近损坏时
        // return self::REQUEUE; // 消息正常，但您希望推迟处理
    }

    public static function getSubscribedTopics()
    {
        return ['aFooTopic'];
    }
}
```

将其注册为容器服务。如果您不使用自动装配，请将其订阅到该主题。

```yaml
foo_message_processor:
    class: 'FooProcessor'
    tags:
        - { name: 'enqueue.topic_subscriber' }
        # 使用下面的变体连接到特定客户端
        # 另外请注意，若不禁用自动配置，则上述标记将自动应用于默认客户端
        # - { name: 'enqueue.topic_subsciber', client: 'some_other_transport' }
```

现在你可以开始消费消息了：

```bash
$ ./bin/console enqueue:consume --setup-broker -vvv
```

您可以选择特定的客户端进行消费：

```bash
$ ./bin/console enqueue:consume --setup-broker --client="some_other_transport" -vvv
```


_**注意**: 添加 `-vvv` 以了解在您消费消息时发生了什么，那里有很多有价值的调试信息。_

[返回首页](index.md)