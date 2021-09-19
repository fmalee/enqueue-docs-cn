---
layout: default
parent: 客户端
title: 扩展
nav_order: 6
---
{% include support.md %}

# 客户端扩展

您可以添加钩子到发送进程中。您必须创建一个实现 `Enqueue\Client\ExtensionInterface` 接口的扩展类。
例如，`TimestampMessageExtension` 扩展在将每条消息发送到 MQ 之前为其添加时间戳。

```php
<?php
namespace Acme;

use Enqueue\Client\ExtensionInterface;
use Enqueue\Client\Message;

class TimestampMessageExtension implements ExtensionInterface
{
    public function onPreSend($topic, Message $message)
    {
        if ($message->getTimestamp()) {
            $message->setTimestamp(time());
        }
    }

    public function onPostSend($topic, Message $message)
    {

    }
}
```

## Symfony

Symfony 中使用该扩展，您必须将其注册为具有特殊标签的容器服务。

```yaml
# config/services.yaml

services:
  timestamp_message_extension:
    class: Acme\TimestampMessageExtension
    tags:
      - { name: 'enqueue.client.extension' }
```

您可以使用数字来添加 `priority` 属性。设置的值越高，调用扩展的时间就越早。

[返回首页](../index.md)