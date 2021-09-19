---
layout: default
parent: "Symfony bundle"
title: Consumption extension
nav_order: 9
---
{% include support.md %}

# 消费扩展

在这里，将展示如何创建自定义扩展并注册它。
让我们首先创建一个扩展本身：

```php
<?php
// src/AppBundle/Enqueue;
namespace AppBundle\Enqueue;

use Enqueue\Consumption\PostMessageReceivedExtensionInterface;
use Enqueue\Consumption\Context\PostMessageReceived;

class CountProcessedMessagesExtension implements PostMessageReceivedExtensionInterface
{
    private $processedMessages = 0;

    public function onPostMessageReceived(PostMessageReceived $context): void
    {
        $this->processedMessages += 1;
    }
}
```

现在我们必须使用特殊标签将其注册为 Symfony 服务：

```yaml
services:
    app.enqueue.count_processed_messages_extension:
        class: 'AppBundle\Enqueue\CountProcessedMessagesExtension'
        tags:
            - { name: 'enqueue.consumption.extension', priority: 10 }
```

在使用多个enqueue实例时，您可以通过提供额外的标签属性来将扩展应用于特定或所有实例：

```yaml
services:
    app.enqueue.count_processed_messages_extension:
        class: 'AppBundle\Enqueue\CountProcessedMessagesExtension'
        tags:
            - { name: 'enqueue.consumption.extension', priority: 10, client: 'all' }
```

[返回目录](index.md)