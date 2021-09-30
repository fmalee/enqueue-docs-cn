---
layout: default
parent: 客户端
title: 代理支持
nav_order: 3
---
{% include support.md %}

# 客户端支持的代理

以下是 Enqueue Client 支持的传输列表：

|       Transport       |                     Package                      |              DSN               |
| :-------------------: | :----------------------------------------------: | :----------------------------: |
|    AMQP, RabbitMQ     |     [enqueue/amqp-ext](../transport/amqp.md)     |        amqp: amqp+ext:         |
|    AMQP, RabbitMQ     | [enqueue/amqp-bunny](../transport/amqp_bunny.md) |       amqp: amqp+bunny:        |
|    AMQP, RabbitMQ     |   [enqueue/amqp-lib](../transport/amqp_lib.md)   | amqp: amqp+lib: amqp+rabbitmq: |
|     Doctrine DBAL     |       [enqueue/dbal](../transport/dbal.md)       |   mysql: pgsql: pdo_pgsql 等   |
|      Filesystem       |         [enqueue/fs](../transport/fs.md)         |        file:///foo/bar         |
|        Gearman        |    [enqueue/gearman](../transport/gearman.md)    |            gearman:            |
|  GPS, Google PubSub   |        [enqueue/gps](../transport/gps.md)        |              gps:              |
|         Kafka         |     [enqueue/rdkafka](../transport/kafka.md)     |             kafka:             |
|        MongoDB        |    [enqueue/mongodb](../transport/mongodb.md)    |            mongodb:            |
|         Null          |       [enqueue/null](../transport/null.md)       |             null:              |
| Pheanstalk, Beanstalk | [enqueue/pheanstalk](../transport/pheanstalk.md) |           beanstalk:           |
|         Redis         |      [enqueue/redis](../transport/redis.md)      |             redis:             |
|      Amazon SQS       |        [enqueue/sqs](../transport/sqs.md)        |              sqs:              |
|    STOMP, RabbitMQ    |      [enqueue/stomp](../transport/stomp.md)      |             stomp:             |
|         WAMP          |       [enqueue/wamp](../transport/wamp.md)       |             wamp:              |

## 传输特性

|      协议      |  权重  |  延迟  |  限期  | 设置代理 | 消息总线 | Heartbeat |
| :------------: | :----: | :----: | :----: | :------: | :------: | :-------: |
|      AMQP      |   否   |   否   |   是   |    是    |    是    |    否     |
| RabbitMQ AMQP  |   是   |   是   |   是   |    是    |    是    |    是     |
| Doctrine DBAL  |   是   |   是   |   否   |    是    |    否    |    否     |
|   Filesystem   |   否   |   否   |   是   |    是    |    否    |    否     |
|    Gearman     |   否   |   否   |   否   |    否    |    否    |    否     |
| Google PubSub  | 未引入 | 未引入 | 未引入 |    是    |  未引入  |    否     |
|     Kafka      |   否   |   否   |   否   |    是    |    否    |    否     |
|    MongoDB     |   是   |   是   |   是   |    是    |    否    |    否     |
|   Pheanstalk   |   是   |   是   |   是   |    否    |    否    |    否     |
|     Redis      |   否   |   是   |   是   |  不需要  |    否    |    否     |
|   Amazon SQS   |   否   |   是   |   否   |    是    |  未引入  |    否     |
|     STOMP      |   否   |   否   |   是   |    否    |   是**   |    否     |
| RabbitMQ STOMP |   是   |   是   |   是   |  是***   |   是**   |    是     |
|      WAMP      |   否   |   否   |   否   |    否    |    否    |    否     |

* \*\* 如果在代理端手动配置了主题（交换），则支持。
* \*\*\* 如果安装了 RabbitMQ 管理插件，则支持。

[返回首页](../index.md)