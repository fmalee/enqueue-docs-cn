---
layout: default
parent: Consumption
title: Message processors
---
{% include support.md %}

# 消息处理器

* [基础](#基础)
* [答复结果](#答复结果)
* [关于异常](#关于异常)
* [示例](#示例)


## 基础

消息处理器是一个实际处理消息的对象，必须返回一个结果状态。下面是例子：

```php
<?php
use Interop\Queue\Processor;
use Interop\Queue\Message;
use Interop\Queue\Context;

class SendMailProcessor implements Processor
{
    public function process(Message $message, Context $context)
    {
        $this->mailer->send('foo@example.com', $message->getBody());

        return self::ACK;
    }
}
```

通过返回`self::ACK`，处理器将告诉代理该消息已被正确处理。

还有其他状态：

* `self::ACK` - 当消息被成功处理并且消息可以从队列中移除时使用这个常量。
* `self::REJECT` - 当消息无效或无法处理时使用此常量。消息将从队列中删除。
* `self::REQUEUE` - 当消息无效或无法立即处理，但我们需要稍后再试时，使用此常量。

查看下一个示例，该示例展示了发送邮件之前的消息验证。如果该消息无效，则处理器将拒绝它。

```php
<?php
use Interop\Queue\Processor;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Enqueue\Util\JSON;

class SendMailProcessor implements Processor
{
    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());
        if ($user  = $this->userRepository->find($data['userId'])) {
            return self::REJECT;
        }

        $this->mailer->send($user->getEmail(), $data['text']);

        return self::ACK;
    }
}
```

有一个 `isRedelivered` 方法可以查明消息先前是否失败。
如果它返回 `true` 则尝试处理该消息。

```php
<?php
use Interop\Queue\Processor;
use Interop\Queue\Message;
use Interop\Queue\Context;

class SendMailProcessor implements Processor
{
    public function process(Message $message, Context $context)
    {
        if ($message->isRedelivered()) {
            return self::REQUEUE;
        }

        $this->mailer->send('foo@example.com', $message->getBody());

        return self::ACK;
    }
}
```

第二个参数是你的上下文。您可以使用它向其他队列\主题发送消息。

```php
<?php
use Interop\Queue\Processor;
use Interop\Queue\Message;
use Interop\Queue\Context;

class SendMailProcessor implements Processor
{
    public function process(Message $message, Context $context)
    {
        $this->mailer->send('foo@example.com', $message->getBody());

        $queue = $context->createQueue('anotherQueue');
        $message = $context->createMessage('Message has been sent');
        $context->createProducer()->send($queue, $message);

        return self::ACK;
    }
}
```

## 答复结果

消费组件提供了一些有用的扩展，例如有一个使处理 RPC 更简单的扩展。
生产者可能会等待消费者的回复，并且为了发送它，处理器必须返回答复结果。
不要忘记添加 `ReplyExtension`。

```php
<?php
use Interop\Queue\Processor;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Enqueue\Consumption\ChainExtension;
use Enqueue\Consumption\QueueConsumer;
use Enqueue\Consumption\Extension\ReplyExtension;
use Enqueue\Consumption\Result;

class SendMailProcessor implements Processor
{
    public function process(Message $message, Context $context)
    {
        $this->mailer->send('foo@example.com', $message->getBody());

        $replyMessage = $context->createMessage('Message has been sent');

        return Result::reply($replyMessage);
    }
}

/** @var \Interop\Queue\Context $context */

$queueConsumer = new QueueConsumer($context, new ChainExtension([
    new ReplyExtension()
]));

$queueConsumer->bind('foo', new SendMailProcessor());

$queueConsumer->consume();
```

## 关于异常

建议不要捕获异常并[快速失败](https://en.wikipedia.org/wiki/Fail-fast)。
还可以考虑使用 [Supervisord](supervisord.org) 或类似的进程管理器来重新启动已退出的消费者。

尽管建议失败，但在某些情况下您可能希望捕获异常。

* 消息验证器对无效消息抛出异常。最好抓住它并返回`REJECT`。
* 某些传输（[Doctrine DBAL](../transport/dbal.md)、[Filesystem](../transport/filesystem.md)、[Redis](../transport/redis.md)）确实注意到一个错误，因此将无法重新传递消息。该消息已完全丢失。您可能希望捕获异常以正确重新传递\重新入队该消息

# 示例

随意贡献你自己的示例。

* [LiipImagineBundle. ResolveCacheProcessor](https://github.com/liip/LiipImagineBundle/blob/713e36f5df353d7c5345daed5a2eefc23c103849/Async/ResolveCacheProcessor.php#L1)
* [EnqueueElasticaBundle. ElasticaPopulateProcessor](https://github.com/php-enqueue/enqueue-elastica-bundle/blob/7c05c55b1667f9cae98325257ba24fc101f87f97/Async/ElasticaPopulateProcessor.php#L1)
* [formapro/pvm. HandleAsyncTransitionProcessor](https://github.com/formapro/pvm/blob/d5e989a77eb1540a93e69abacc446b3d7937292d/src/Enqueue/HandleAsyncTransitionProcessor.php#L1)
* [php-quartz. EnqueueRemoteTransportProcessor](https://github.com/php-quartz/quartz-dev/blob/91690aa535b0322510b4555dab59d6ae9d7044e5/pkg/bridge/Enqueue/EnqueueRemoteTransportProcessor.php#L1)
* [php-comrade. CreateJobProcessor](https://github.com/php-comrade/comrade-dev/blob/43c0662b74340aae318bceb15d8564670325dcee/apps/jm/src/Queue/CreateJobProcessor.php#L1)
* [prooph/psb-enqueue-producer. EnqueueMessageProcessor](https://github.com/prooph/psb-enqueue-producer/blob/c80914a4092b42b2d0a7ba698b216e0af23bab42/src/EnqueueMessageProcessor.php#L1)

[返回目录](../index.md)

