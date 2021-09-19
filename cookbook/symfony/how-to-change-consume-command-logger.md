---
layout: default
nav_exclude: true
---
{% include support.md %}

# 如何修改消费命令日志器

默认情况下，`bin/console enqueue:consume`（或 `bin/console enqueue:transport:consume`）命令会将消息打印到输出。
信息量可以由冗余度选项（-v、-vv、-vvv）控制。

为了更改命令使用的默认日志器，您必须在默认记录器之前注册一个 `LoggerExtension`。
该扩展要求您提供日至期器服务，因此只需传递您要使用的服务即可。这是您如何做的：

```yaml
// config/services.yaml

services:
    app_logger_extension:
        class: 'Enqueue\Consumption\Extension\LoggerExtension'
        public: false
        arguments: ['@logger']
        tags:
            - { name: 'enqueue.consumption.extension', priority: 255 }

```

具有最高优先级的日志器扩展将会被设置为记录器。

[返回目录](../../index.md)



