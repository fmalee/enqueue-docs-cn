---
layout: default
parent: "Symfony bundle"
title: Job queue
nav_order: 8
---
{% include support.md %}

# 作业

当您的消息流具有多个一个接一个运行的步骤（任务）时，请使用作业。
作业也可以保证该作业是独一无二的，即在上一份作业完成之前，您不能开始同名的新作业。

* [安装](#安装)
* [唯一性作业](#唯一性作业)
* [子作业](#子作业)
* [依赖性作业](#依赖性作业)

## 安装

安装 Enqueue 作业队列的最简单方法是请求一个 `enqueue/job-queue-pack` 包。
它会安装好您需要的一切。如果您使用 Symfony Flex，它还可以为您配置好所有东西。

```bash
$ composer require enqueue/job-queue-pack=^0.8
```

_**注意**：只要您使用 Symfony Flex，你就完成安装了。如果没有，请继续阅读安装章节。_

* 注册已安装的包

```php
<?php
// app/AppKernel.php

use Symfony\Component\HttpKernel\Kernel;

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = [
            new Doctrine\Bundle\DoctrineBundle\DoctrineBundle(),
            new Enqueue\Bundle\EnqueueBundle(),
        ];

        return $bundles;
    }
}
````

* 配置已安装的包：

```yaml
# app/config/config.yml

enqueue:
    default:
        # 添加基础的包配置

        job: true
        
        # 将包的默认作业实体映射添加到应用的实体管理器。
        # 要将自己的映射实体用于作业时，请将其设置为false。
        default_mapping: true

doctrine:
    # 添加基础的包配置

    orm:
        mappings:
            EnqueueJobQueue:
                is_bundle: false
                type: xml
                dir: '%kernel.project_dir%/vendor/enqueue/job-queue/Doctrine/mapping'
                prefix: 'Enqueue\JobQueue\Doctrine\Entity'

```

* 运行doctrine schema更新命令：

```bash
$ bin/console doctrine:schema:update
```

## 唯一性作业

确保一次只运行一个具有此名称的作业。
例如，您有一个构建搜索索引的任务。
这需要相当多的时间，而且您不希望同一任务的另一个实例同时工作。这是如何做到的：

* 编写一个作业处理器类：

```php
<?php
namespace App\Queue;

use Interop\Queue\Message;
use Interop\Queue\Processor;
use Interop\Queue\Context;
use Enqueue\Util\JSON;
use Enqueue\JobQueue\JobRunner;
use Enqueue\JobQueue\Job;
use Enqueue\Client\CommandSubscriberInterface;

class SearchReindexProcessor implements Processor, CommandSubscriberInterface
{
    private $jobRunner;

    public function __construct(JobRunner $jobRunner)
    {
        $this->jobRunner = $jobRunner;
    }

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runUnique(
            $message->getMessageId(),
            'search:index:reindex',
            function (JobRunner $runner, Job $job) use ($data) {
                // 完成你的作业

                return true; // ACK 该消息，false则为 REJECT
            }
        );

        return $result ? self::ACK : self::REJECT;
    }

    public static function getSubscribedCommand()
    {
        return 'search_reindex';
    }
}
```

* 注册它

```yaml
services:
  app_queue_search_reindex_processor:
    class: 'App\Queue\SearchReindexProcessor'
    arguments: ['@Enqueue\JobQueue\JobRunner']
    tags:
        - { name: 'enqueue.command_subscriber' }
```

* 调度命令

```php
<?php
use Symfony\Component\DependencyInjection\ContainerInterface;
use Enqueue\Client\ProducerInterface;

/** @var ContainerInterface $container  */

$producer = $container->get(ProducerInterface::class);

$producer->sendCommand('search_reindex');
```

## 子作业

并行运行多个子作业。步骤与我们上面的描述相同。

```php
<?php
use Enqueue\JobQueue\JobRunner;
use Enqueue\JobQueue\Job;
use Enqueue\Client\ProducerInterface;
use Enqueue\Util\JSON;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Interop\Queue\Processor;

class Step1Processor implements Processor
{
    /**
     * @var JobRunner
     */
    private $jobRunner;

    /**
     * @var ProducerInterface
     */
    private $producer;

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runUnique(
            $message->getMessageId(),
            'search:index:reindex',
            function (JobRunner $runner, Job $job) use ($data) {
                // 例如，第一步为第二步生成任务

                foreach ($entities as $entity) {
                    // 每个作业名称必须是唯一的
                    $jobName = 'search:index:index-single-entity:' . $entity->getId();
                    $runner->createDelayed(
                        $jobName,
                        function (JobRunner $runner, Job $childJob) use ($entity) {
                            $this->producer->sendEvent('search:index:index-single-entity', [
                                'entityId' => $entity->getId(),
                                'jobId' => $childJob->getId(),
                            ]);
                    });
                }

                return true; // ACK 该消息，false则为 REJECT
            }
        );

        return $result ? self::ACK : self::REJECT;
    }
}

class Step2Processor implements Processor
{
    /**
     * @var JobRunner
     */
    private $jobRunner;

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runDelayed(
            $data['jobId'],
            function (JobRunner $runner, Job $job) use ($data) {
                // 完成你的作业

                return true; // ACK 该消息，false则为 REJECT
            }
        );

        return $result ? self::ACK : self::REJECT;
    }
}
```

## 依赖性作业

当您的作业流程有多个步骤，但您想在所有步骤完成后立即发送新消息时，请使用依赖性作业。
步骤与我们上面的描述相同。

```php
<?php
use Enqueue\JobQueue\JobRunner;
use Enqueue\JobQueue\Job;
use Enqueue\JobQueue\DependentJobService;
use Enqueue\Util\JSON;
use Interop\Queue\Message;
use Interop\Queue\Context;
use Interop\Queue\Processor;

class ReindexProcessor implements Processor
{
    /**
     * @var JobRunner
     */
    private $jobRunner;

    /**
     * @var DependentJobService
     */
    private $dependentJob;

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runUnique(
            $message->getMessageId(),
            'search:index:reindex',
            function (JobRunner $runner, Job $job) use ($data) {
                // 注册两个依赖性作业
                // 将在该作业和所有子作业都完成时发送下一条消息到队列
                $context = $this->dependentJob->createDependentJobContext($job->getRootJob());
                $context->addDependentJob('topic1', 'message1');
                $context->addDependentJob('topic2', 'message2');

                $this->dependentJob->saveDependentJob($context);

                // 完成你的作业

                return true; // ACK 该消息，false则为 REJECT
            }
        );

        return $result ? self::ACK : self::REJECT;
    }
}
```

[返回目录](index.md)