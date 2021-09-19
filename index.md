---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Index
nav_order: 0
---

{% include support.md %}

## 文档.

* [快速指南](quick_tour.md)
* [关键概念](concepts.md)
* [传输](#transports)
    - 基于[ext](transport/amqp.md)、[bunny](transport/amqp_bunny.md)、[lib](transport/amqp_lib.md)的Amqp
    - [Amazon SNS-SQS](transport/snsqs.md)
    - [Amazon SQS](transport/sqs.md)
    - [Google PubSub](transport/gps.md)
    - [Beanstalk (Pheanstalk)](transport/pheanstalk.md)
    - [Gearman](transport/gearman.md)
    - [Kafka](transport/kafka.md)
    - [Stomp](transport/stomp.md)
    - [Redis](transport/redis.md)
    - [Wamp](transport/wamp.md)
    - [Doctrine DBAL](transport/dbal.md)
    - [Filesystem](transport/filesystem.md)
    - [Null](transport/null.md)
* [消费](#consumption)
    - [扩展](consumption/extensions.md)
    - [消息处理器](consumption/message_processor.md)
* [客户端](#client)
    - [快速指南](client/quick_tour.md)
    - [消息示例](client/message_examples.md)
    - [支持的代理](client/supported_brokers.md)
    - [消息总线](client/message_bus.md)
    - [RPC调用](client/rpc_call.md)
    - [扩展](client/extensions.md)
* [作业队列](#job-queue)
    - [运行唯一性作业](job_queue/run_unique_job.md)
    - [运行子作业](job_queue/run_sub_job.md)
* [Symfony Bundle](bun[返回目录](../index.md))
    - [快速指南](bundle/quick_tour.md)
    - [配置参考](bundle/config_reference.md)
    - [Cli命令](bundle/cli_commands.md)
    - [消息生产者](bundle/message_producer.md)
    - [消息处理器](bundle/message_processor.md)
    - [异步事件](bundle/async_events.md)
    - [异步命令](bundle/async_commands.md)
    - [作业队列](bundle/job_queue.md)
    - [消费扩展](bundle/consumption_extension.md)
    - [生产设置](bundle/production_settings.md)
    - [调试](bundle/debugging.md)
    - [功能测试](bundle/functional_testing.md)
* [Laravel](#laravel)
    - [快速指南](laravel/quick_tour.md)
    - [队列](laravel/queues.md)
* [Magento](#magento)
    - [快速指南](magento/quick_tour.md)
    - [Cli命令](magento/cli_commands.md)
* [Magento2](#magento2)
    - [快速指南](magento2/quick_tour.md)
    - [Cli命令](magento2/cli_commands.md)
* [Yii](#yii)
    - [AMQP交互驱动](yii/amqp_driver.md)
* [EnqueueElasticaBundle 概述](elastica-bundle/overview.md)
* [DSN解析器](dsn.md)
* [监控](monitoring.md)
* [用例](#use-cases)
    - [Symfony：异步事件调度器](async_event_dispatcher/quick_tour.md)
    - [Monolog：发送消息到消息队列](monolog/send-messages-to-mq.md)
* [开发](#development)
    - [贡献](contribution.md)

## 进阶

* [Symfony](#symfony-cookbook)
    - [如何修改消费命令日志器](cookbook/symfony/how-to-change-consume-command-logger.md)

## 博客

* [PHP RabbitMQ 入门](https://blog.forma-pro.com/getting-started-with-rabbitmq-in-php-84d331e20a66)
* [在 Symfony 中开始使用 RabbitMQ](https://blog.forma-pro.com/getting-started-with-rabbitmq-in-symfony-cb06e0b674f1)
* [从 RabbitMqBundle 迁移到 EnqueueBundle 的缘由和方法](https://blog.forma-pro.com/the-how-and-why-of-the-migration-from-rabbitmqbundle-to-enqueuebundle-6c4054135e2b)
* [RabbitMQ 重新投递陷阱](https://blog.forma-pro.com/rabbitmq-redelivery-pitfalls-440e0347f4e0)
* [RabbitMQ 消息延迟传递](https://blog.forma-pro.com/rabbitmq-delayed-messaging-da802e3a0aa9)
* [基于AMQP教育的RabbitMQ教程](https://blog.forma-pro.com/rabbitmq-tutorials-based-on-amqp-interop-cf325d3b4912)
* [LiipImagineBundle：在后台处理图像](https://blog.forma-pro.com/liipimaginebundle-process-images-in-background-3838c0ed5234)
* [FOSElasticaBundle：提高 fos:elastica:populate 命令的性能](https://github.com/php-enqueue/enqueue-elastica-bundle)
* [应用消息总线到每个PHP应用](https://blog.forma-pro.com/message-bus-to-every-php-application-42a7d3fbb30b)
* [Symfony异步事件调度](https://blog.forma-pro.com/symfony-async-eventdispatcher-d01055a255cf)
* [将 Swiftmailer 电邮假脱机到真正的消息队列](https://blog.forma-pro.com/spool-swiftmailer-emails-to-real-message-queue-9ecb8b53b5de)
* [Yii PHP 框架采用了 AMQP 交互](https://blog.forma-pro.com/yii-php-framework-has-adopted-amqp-interop-85ab47c9869f)
* [(En)queue Symfony 控制台命令](http://tech.yappa.be/enqueue-symfony-console-commands)
* [通过 Symfony Messenger 从 RabbitMq 迁移到 PhpEnqueue](https://medium.com/@stefanoalletti_40357/from-rabbitmq-to-phpenqueue-via-symfony-messenger-b8260d0e506c)

## 为本文档做出贡献

要在本地运行此文档，您可以在本地计算机上创建 Jekyll 环境或使用 docker 容器。
要运行 docker 容器，您可以使用来自仓库根目录的命令：

```shell
docker run -p 4000:4000 --rm --volume="${PWD}/docs:/srv/jekyll" -it jekyll/jekyll jekyll serve --watch
```
一旦构建完成，文档将可从 http://localhost:4000/ 访问，并在更改时自动重建。
