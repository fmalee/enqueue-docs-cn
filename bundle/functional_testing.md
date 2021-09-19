---
layout: default
parent: "Symfony bundle"
title: Functional testing
nav_order: 12
---
{% include support.md %}

# 功能测试

本章我们就如何测试消息队列相关的逻辑给出一些建议。

* [NULL传输](#NULL传输)
* [可追溯的消息生产者](#可追溯的消息生产者)

## NULL传输

在测试应用时，您通常不需要向真正的代理发送真实的消息。
或者甚至依赖于 MQ 代理。
这正是 NULL 传输的目的。
当您要求它发送消息时，它很简单的什么都不做。
这在测试中非常有用。
这是您可以配置它的方法。

```yaml
# app/config/config_test.yml

enqueue:
    default:
        transport: 'null:'
        client: ~
```

## 可追溯的消息生产者

想象一下，您有一个具有在内部发送消息的 `someMethod()` 方法的 `my_service` 服务，然后您必须确认消息是否已发送。
有一个解决方案。您必须在测试环境中启用可追溯的消息生产者。

```yaml
# app/config/config_test.yml

enqueue:
    default:
        client:
            traceable_producer: true
```

如果你这样做了，你可以使用它的 `getTraces`、`getTopicTraces` 或者 `clearTraces` 方法。下面是一个例子：

```php
<?php
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Enqueue\Client\TraceableProducer;

class FooTest extends WebTestCase
{
    /** @var  \Symfony\Bundle\FrameworkBundle\Client */
    private $client;

    public function setUp(): void
    {
        $this->client = static::createClient();
    }

    public function testMessageSentToFooTopic()
    {
        // 在此处使用您自己的业务逻辑：
        $service = $this->client->getContainer()->get('my_service');

        // someMethod() 是您业务逻辑的一部分，并且它正在某处调用 $producer->send('fooTopic', 'messageBody');
        $service->someMethod();

        $traces = $this->getProducer()->getTopicTraces('fooTopic');

        $this->assertCount(1, $traces);
        $this->assertEquals('messageBody', $traces[0]['message']);
    }

    /**
     * @return TraceableProducer
     */
    private function getProducer()
    {
        return $this->client->getContainer()->get(TraceableProducer::class);
    }
}
```

[返回目录](index.md)