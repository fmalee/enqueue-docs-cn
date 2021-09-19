---
layout: default
parent: "Symfony bundle"
title: Production settings
nav_order: 10
---
{% include support.md %}

# 生产设置

## Supervisord

正如您在[快速指南](quick_tour.md)中所读到的那样，您必须运行 `enqueue:consume` 才能处理消息。
 PHP进程并不是为长时间工作而设计的，所以它必须定期退出。
或者，该命令可能因错误或异常而退出。
必须有东西把它带回并继续消息的消费。
为此，我们建议您使用 [Supervisord](http://supervisord.org/)。
它会启动进程，并在它们工作时密切关注。

这里是一个 Supervisord 配置的示例。
它同时运行四个 `enqueue:consume` 命令实例。

```ini
[program:pf_message_consumer]
command=/path/to/bin/console --env=prod --no-debug --time-limit="now + 5 minutes" enqueue:consume
process_name=%(program_name)s_%(process_num)02d
numprocs=4
autostart=true
autorestart=true
startsecs=0
user=apache
redirect_stderr=true
```

_**注意**：`--time-limit` 是在告诉命令要在 5分钟后退出。_

[返回目录](index.md)