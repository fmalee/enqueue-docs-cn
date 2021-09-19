---
layout: default
parent: "Symfony bundle"
title: Config reference
nav_order: 2
---
{% include support.md %}

# 配置参考

您可以通过运行 `./bin/console config:dump-reference enqueue` 命令来获取这些信息。

```yaml
# 别名为“enqueue”的扩展的默认配置
enqueue:

    # 原型
    key:

        # 传输选项可以接受字符串DSN、带有DSN密钥的数组或null。它接受额外的选项。要了解可以设置什么选项，请查看连接工厂的构造函数的文档块。
        transport:            # 必须

            # MQ代理的DSN。支持一下模式："file"、"amqp"、"amqps"、"db2"、"ibm-db2"、"mssql"、"sqlsrv"、"mysql"、"mysql2"、"pgsql"、"postgres"、"sqlite"、"sqlite3"、"null"、"gearman"、"beanstalk"、"kafka"、"rdkafka"、"redis"、"rediss"、"stomp"、"sqs"、"gps"、"mongodb"、"wamp"、"ws"。为了使用"file",、"amqp"、"amqps"、"db2"、"ibm-db2"、"mssql"、"sqlsrv"、"mysql"、"mysql2"、"pgsql"、"postgres"、"sqlite"、"sqlite3"、"null"、"gearman"、"beanstalk"、"kafka"、"rdkafka"、"redis"、"rediss"、"stomp"、"sqs"、"gps"、"mongodb"、"wamp"、"ws"，你必须安装一个包。
            dsn:                  ~ # 必须

            # 实现了 Interop\Queue\ConnectionFactory 接口的连接工厂类
            connection_factory_class: ~

            # 实现了 Enqueue\ConnectionFactoryFactoryInterface 接口的工厂类
            factory_service:      ~

            # 实现了 Enqueue\ConnectionFactoryFactoryInterface 接口的工厂服务类
            factory_class:        ~
        consumption:

            # 队列消费者等待一个消息的时间（毫秒），默认为100毫秒。
            receive_timeout:      10000
        client:
            traceable_producer:   true
            prefix:               enqueue
            separator:            .
            app_name:             app
            router_topic:         default
            router_queue:         default
            router_processor:     null
            redelivered_delay_time: 0
            default_queue:        default

            # 包含驱动特定选项的数组
            driver_options:       []

        # 监视选项可以接受字符串DSN、带有DSN密钥的数组或null。它接受额外的选项。要了解可以设置什么选项，请查看stats storage构造函数的文档块。
        monitoring:

            # 统计数据存储DSN。支持这些方案：“wamp”、“ws”、“XDB”。
            dsn:                  ~ # 必须

            # 实现了 Enqueue\Monitoring\StatsStorageFactory 接口的工厂类
            storage_factory_service: ~

            # 实现了 Enqueue\Monitoring\StatsStorageFactory 接口的工厂服务类
            storage_factory_class: ~
        async_commands:
            enabled:              false
            timeout:              60
            command_name:         ~
            queue_name:           ~
        job:
            enabled:              false
            default_mapping:      true
        async_events:
            enabled:              false
        extensions:
            doctrine_ping_connection_extension: false
            doctrine_clear_identity_map_extension: false
            doctrine_odm_clear_identity_map_extension: false
            doctrine_closed_entity_manager_extension: false
            reset_services_extension: false
            signal_extension:     true
            reply_extension:      true
```

[返回目录](index.md)