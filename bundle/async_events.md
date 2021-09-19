---
layout: default
parent: "Symfony bundle"
title: Async events
nav_order: 6
---
{% include support.md %}

# 异步事件

EnqueueBundle 允许您异步调度事件。
在幕后，它将您的监听器替换为向 MQ 发送消息的监听器。
该消息包含事件对象。
消费者一旦收到消息，就会恢复该事件并将其调度给异步监听器。

异步监听器的好处：

* 减少响应时间。工作被推迟到消费者进程。
* 更好的容错能力。异步监听器中的错误不会影响用户。消息将等到您修复错误为止。
* 更好的弹性。添加更多消费者以满足负载。

_**注意**：在 Symfony 3.0 之前，事件会包含 `eventDispatcher`，并且默认的 php 序列化转换器无法序列化对象。每个异步事件都应该注册一个转换器。阅读[事件转换器](#事件转换器)。_

## 配置

Symfony 事件当前是同步处理的，启用 EnqueueBundle 的异步配置会导致标记的监听器异步的延迟对消费者的操作。
如果您已经[安装了该包](quick_tour.md#install)，则启用 `async_events`。

```yaml
# app/config/config.yml

enqueue:
    default:
        async_events:
            enabled: true
            # 如果您想使用终端上发送消息，请使用spool_producer（它进一步缩短了响应时间）:
            # spool_producer: true
```

## 用例

为了使您的监听器异步，您必须向 `kernel.event_listener` 标签添加 `async: true` 属性，如下所示：

```yaml
# app/config/config.yml

services:
    acme.foo_listener:
        class: 'AcmeBundle\Listener\FooListener'
        tags:
            - { name: 'kernel.event_listener', async: true, event: 'foo', method: 'onEvent' }
```

或注册到 `kernel.event_subscriber`：

```yaml
# app/config/config.yml

services:
    test_async_subscriber:
        class: 'AcmeBundle\Listener\TestAsyncSubscriber'
        tags:
            - { name: 'kernel.event_subscriber', async: true }
```

基本上就是这样。文档的其余部分将描述高级功能。

## 高级用例

您还可以直接添加异步监听器并为其注册自定义消息处理器：

```yaml
# app/config/config.yml

services:
    acme.async_foo_listener:
        class: 'Enqueue\AsyncEventDispatcher\AsyncListener'
        public: false
        arguments: ['@enqueue.transport.default.context', '@enqueue.events.registry', 'a_queue_name']
        tags:
          - { name: 'kernel.event_listener', event: 'foo', method: 'onEvent' }
```


## 事件转换器

默认情况下，bundle 使用[php 序列化](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue-bundle/Events/PhpSerializerEventTransformer.php)转换器来通过 MQ 传递事件。
您可以通过实现 `Enqueue\AsyncEventDispatcher\EventTransformer` 接口来为每种事件类型编写转换器。
考虑下一个例子。它展示了如何发送包含 Doctrine 实体作为主题的事件：

```php
<?php
namespace AcmeBundle\Listener;

// src/AcmeBundle/Listener/FooEventTransformer.php

use Enqueue\Client\Message;
use Enqueue\Consumption\Result;
use Interop\Queue\Message as QueueMessage;
use Enqueue\Util\JSON;
use Symfony\Component\EventDispatcher\Event;
use Enqueue\AsyncEventDispatcher\EventTransformer;
use Doctrine\Bundle\DoctrineBundle\Registry;
use Symfony\Component\EventDispatcher\GenericEvent;

class FooEventTransformer implements EventTransformer
{
    /** @var Registry @doctrine */
    private $doctrine;

    public function __construct(Registry $doctrine)
    {
        $this->doctrine = $doctrine;
    }

    /**
     * {@inheritdoc}
     *
     * @param GenericEvent $event
     */
    public function toMessage($eventName, Event $event = null)
    {
        $entity = $event->getSubject();
        $entityClass = get_class($entity);

        $manager = $this->doctrine->getManagerForClass($entityClass);
        $meta = $manager->getClassMetadata($entityClass);

        $id = $meta->getIdentifierValues($entity);

        $message = new Message();
        $message->setBody([
            'entityClass' => $entityClass,
            'entityId' => $id,
            'arguments' => $event->getArguments()
        ]);

        return $message;
    }

    /**
     * {@inheritdoc}
     */
    public function toEvent($eventName, QueueMessage $message)
    {
        $data = JSON::decode($message->getBody());

        $entityClass = $data['entityClass'];

        $manager = $this->doctrine->getManagerForClass($entityClass);
        if (false == $entity = $manager->find($entityClass, $data['entityId'])) {
            return Result::reject('The entity could not be found.');
        }

        return new GenericEvent($entity, $data['arguments']);
    }
}
```

并注册它：

```yaml
# app/config/config.yml

services:
    acme.foo_event_transformer:
        class: 'AcmeBundle\Listener\FooEventTransformer'
        arguments: ['@doctrine']
        tags:
            - {name: 'enqueue.event_transformer', eventName: 'foo' }
```

该 `eventName` 属性接受正则表达式。
你可以做使用`eventName: '/foo\..*?/'`。
它将此转换器用于名称以 `foo.` 开头的所有事件。

[返回目录](index.md)