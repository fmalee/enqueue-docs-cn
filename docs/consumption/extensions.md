---
layout: default
parent: Consumption
title: Extensions
---
{% include support.md %}

# 消费扩展

您可以在[快速指南](../quick_tour.md#消费)中了解如何注册扩展。
有专门的[章节](../bundle/consumption_extension.md)介绍如何在 Symfony 应用中添加扩展。

## [LoggerExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/LoggerExtension.php)

它将日志器设置为对消费者上下文进行队列。所有日志消息都将转向它。

## [DoctrineClearIdentityMapExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue-bundle/Consumption/Extension/DoctrineClearIdentityMapExtension.php)

它在处理消息后清除 Doctrine 的身份映射。它减少了内存使用。

## [DoctrinePingConnectionExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue-bundle/Consumption/Extension/DoctrinePingConnectionExtension.php)

它会测试数据库连接，如果连接丢失了它会重新连接。用于修复"MySQL has gone away"错误。

## [DoctrineClosedEntityManagerExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue-bundle/Consumption/Extension/DoctrineClosedEntityManagerExtension.php)

如果实体管理器已关闭，则该扩展会中断消费。

## [ResetServicesExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue-bundle/Consumption/Extension/ResetServicesExtension.php)

它重置所有带有“kernel.reset”标签的服务。
例如，这包括所有的 Monolog 日志器（如果已安装），并将刷新/清理所有缓冲区，重置内部状态，并将它们恢复到可以再次接收日志记录的状态。

## [ReplyExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/ReplyExtension.php)

它带有 RPC 代码并简化了答复逻辑。
它负责向答复队列发送答复消息。

## [SetupBrokerExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Client/ConsumptionExtension/SetupBrokerExtension.php)

它负责在代理端配置所有内容。队列、主题、绑定等。
启用 `--setup-broker` 选项后，扩展将在运行时中添加。

## [LimitConsumedMessagesExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/LimitConsumedMessagesExtension.php)

该扩展对处理的消息进行计数，一旦达到限制，它就会中断消费。
启用 `--message-limit=10` 选项后，该扩展在运行时中添加。

## [LimitConsumerMemoryExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/LimitConsumerMemoryExtension.php)

一旦达到内存限制，该扩展就会中断消费。
启用 `--memory-limit=512` 选项后，该扩展在运行时中添加。
该值为 Mb级别。

## [LimitConsumptionTimeExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/LimitConsumptionTimeExtension.php)

一旦达到时间限制，该扩展会中断消费。
启用 `--time-limit="now + 2 minutes"` 选项后，该扩展在运行时中添加。

## [SignalExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Consumption/Extension/SignalExtension.php)

该扩展会捕获进程信号并优雅地停止消费。
仅适用于 NIX 平台。

## [DelayRedeliveredMessageExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/pkg/enqueue/Client/ConsumptionExtension/DelayRedeliveredMessageExtension.php)

该扩展检查接收到的消息是否已重新投递（曾尝试处理该消息但失败）状态。
如果是这样，该扩展会拒绝原始消息，并创建一个带有延迟的复制消息。

## [ConsumerMonitoringExtension](https://github.com/php-enqueue/enqueue-dev/blob/master/docs/monitoring.md#consumption-extension)

Enqueue QueueConsumer 有一个 ConsumerMonitoringExtension 扩展。
它可以为你收集已消费消息和消费者统计信息，并将它们发送到 Grafana、InfluxDB 或 Datadog。

[返回目录](../index.md)