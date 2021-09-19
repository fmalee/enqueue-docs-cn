---
layout: default
title: Contribution
nav_order: 99
---

{% include support.md %}

# 贡献

要做出贡献，您必须向 [enqueue-dev](https://github.com/php-enqueue/enqueue-dev) 仓库发起拉取请求。
只读子树拆分的[仓库](https://github.com/php-enqueue/enqueue-dev/blob/master/bin/subtree-split#L46)的拉取请求将被关闭。

## 设置环境

```
composer install
./bin/pre-commit -i
./bin/dev -b
```

完成后，您可以处理功能或错误修复。

如果需要，您还可以使用 Composer 脚本来运行代码格式化和静态分析：
* 对于代码格式化，运行 `composer run cs-lint`。（可选）添加文件名：例如 `composer run cs-lint pkg/null/NullTopic.php`。
* 您还可以使用 `composer run cs-fix` 来修复代码格式。
* 静态代码分析可以使用 `composer run phpstan`。如上所述，您可以传递特定文件。

## 测试

运行测试

```
./bin/test.sh
```

或仅适用于包：


```
./bin/test.sh pkg/enqueue
```

## 提交

当您尝试提交更改时，请允许`php-cs-fixer`。它修复了所有编码风格问题。不要忘记暂存它们并提交所有内容。
一切完成后，在官方仓库上打开拉取请求。

## 什么鬼？

* 如果你得到 `rabbitmqssl: forward host lookup failed: Unknown host, wait for service rabbitmqssl:5671`，请执行 `docker-compose down`。

[返回目录](index.md)