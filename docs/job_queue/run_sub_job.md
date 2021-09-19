---
layout: default
parent: 作业队列
title: 运行子作业
nav_order: 2
---
{% include support.md %}

## 运行子作业

这里展示了如何创建和运行单独执行的子作业。
您可以根据需要创建任意数量的子作业。
它们将并行执行。

```php
<?php
use Enqueue\Client\ProducerInterface;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Interop\Queue\Processor;
use Enqueue\JobQueue\JobRunner;
use Enqueue\JobQueue\Job;
use Enqueue\Util\JSON;

class RootJobProcessor implements Processor
{
    /** @var JobRunner */
    private $jobRunner;

    /** @var  ProducerInterface */
    private $producer;

    public function process(Message $message, Context $context)
    {
        $result = $this->jobRunner->runUnique($message->getMessageId(), 'aJobName', function (JobRunner $runner) {
            $runner->createDelayed('aSubJobName1', function (JobRunner $runner, Job $childJob) {
                $this->producer->sendEvent('aJobTopic', [
                    'jobId' => $childJob->getId(),
                    // 子作业要求的其他数据
                ]);
            });

            return true;
        });

        return $result ? self::ACK : self::REJECT;
    }
}

class SubJobProcessor implements Processor
{
    /** @var JobRunner */
    private $jobRunner;

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runDelayed($data['jobId'], function () use ($data) {
            // 处理你的作业

            return true;
        });

        return $result ? self::ACK : self::REJECT;
    }
}
```

[返回首页](../index.md)