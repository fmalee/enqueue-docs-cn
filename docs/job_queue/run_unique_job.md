---
layout: default
parent: Job Queue
title: Run unique job
nav_order: 1
---
{% include support.md %}

## 运行唯一性作业

有一个构建在传输之上的作业队列组件。它提供了一些额外功能：

* 将作业存储到数据库。因此，您可以查询该信息并为其构建 UI。
* 运行唯一性作业。如果使用，则保证没有任何同名作业同时运行。
* 子作业。如果使用，则允许将大作业拆分为较小的部分，并异步并行处理它们。
* 依赖性作业。如果使用，则允许在整个作业（包括子作业）完成时发送消息。

这里有一些例子。
它展示了如何使用作业队列运行唯一性作业（其配置则在专门的章节中阐述）。

```php
<?php
use Interop\Queue\Message;
use Interop\Queue\Context;
use Interop\Queue\Processor;
use Enqueue\JobQueue\JobRunner;

class UniqueJobProcessor implements Processor
{
    /** @var JobRunner */
    private $jobRunner;

    public function process(Message $message, Context $context)
    {
        $result = $this->jobRunner->runUnique($message->getMessageId(), 'aJobName', function () {
            // 进行你的作业，这里没有任何其他进程执行相同的作业。

            return true; // ACK 该消息，false则为 REJECT
        });

        return $result ? self::ACK : self::REJECT;
    }
}
```

[返回目录](../index.md)