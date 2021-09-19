---
layout: default
parent: Client
title: Supported brokers
nav_order: 3
---
{% include support.md %}

# 客户端支持的代理

以下是 Enqueue Client 支持的传输列表：

| Transport             | Package                                                    |  DSN                            |
|:---------------------:|:----------------------------------------------------------:|:-------------------------------:|
| AMQP, RabbitMQ        | [enqueue/amqp-ext](../transport/amqp.md)                   | amqp: amqp+ext:                 |
| AMQP, RabbitMQ        | [enqueue/amqp-bunny](../transport/amqp_bunny.md)           | amqp: amqp+bunny:               |
| AMQP, RabbitMQ        | [enqueue/amqp-lib](../transport/amqp_lib.md)               | amqp: amqp+lib: amqp+rabbitmq:  |
| Doctrine DBAL         | [enqueue/dbal](../transport/dbal.md)                       | mysql: pgsql: pdo_pgsql etc     |
| Filesystem            | [enqueue/fs](../transport/fs.md)                           | file:///foo/bar                 |
| Gearman               | [enqueue/gearman](../transport/gearman.md)                 | gearman:                        |
| GPS, Google PubSub    | [enqueue/gps](../transport/gps.md)                         | gps:                            |
| Kafka                 | [enqueue/rdkafka](../transport/kafka.md)                   | kafka:                          |
| MongoDB               | [enqueue/mongodb](../transport/mongodb.md)                 | mongodb:                        |
| Null                  | [enqueue/null](../transport/null.md)                       | null:                           |
| Pheanstalk, Beanstalk | [enqueue/pheanstalk](../transport/pheanstalk.md)           | beanstalk:                      |
| Redis                 | [enqueue/redis](../transport/redis.md)                     | redis:                          |
| Amazon SQS            | [enqueue/sqs](../transport/sqs.md)                         | sqs:                            |
| STOMP, RabbitMQ       | [enqueue/stomp](../transport/stomp.md)                     | stomp:                          |
| WAMP                  | [enqueue/wamp](../transport/wamp.md)                       | wamp:                           |

## 传输特性

|      协议      |   权重   |   延迟   |   限期   |  设置代理  | 消息总线 | Heartbeat |
| :------------: | :------: | :------: | :------: | :--------: | :------: | :-------: |
|      AMQP      |    No    |    No    |   Yes    |    Yes     |   Yes    |    No     |
| RabbitMQ AMQP  |   Yes    |   Yes    |   Yes    |    Yes     |   Yes    |    Yes    |
| Doctrine DBAL  |   Yes    |   Yes    |    No    |    Yes     |    No    |    No     |
|   Filesystem   |    No    |    No    |   Yes    |    Yes     |    No    |    No     |
|    Gearman     |    No    |    No    |    No    |     No     |    No    |    No     |
| Google PubSub  | Not impl | Not impl | Not impl |    Yes     | Not impl |    No     |
|     Kafka      |    No    |    No    |    No    |    Yes     |    No    |    No     |
|    MongoDB     |   Yes    |   Yes    |   Yes    |    Yes     |    No    |    No     |
|   Pheanstalk   |   Yes    |   Yes    |   Yes    |     No     |    No    |    No     |
|     Redis      |    No    |   Yes    |   Yes    | Not needed |    No    |    No     |
|   Amazon SQS   |    No    |   Yes    |    No    |    Yes     | Not impl |    No     |
|     STOMP      |    No    |    No    |   Yes    |     No     |  Yes**   |    No     |
| RabbitMQ STOMP |   Yes    |   Yes    |   Yes    |   Yes***   |  Yes**   |    Yes    |
|      WAMP      |    No    |    No    |    No    |     No     |    No    |    No     |

* \*\* 如果在代理端手动配置了主题（交换），则可能。
* \*\*\* 如果安装了 RabbitMQ 管理插件，则可行。

[返回目录](../index.md)