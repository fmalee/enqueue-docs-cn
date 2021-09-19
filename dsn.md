---
layout: default
title: DSN Parser
nav_order: 92
---
{% include support.md %}

## DSN 解析器

[Enqueue/dsn](https://github.com/php-enqueue/dsn) 工具有助于解析 DSN\URI 字符串。
Enqueue 传输就是使用该工具来解析 DSN的。

## 安装

```bash
$ composer req enqueue/dsn 0.9.x
```

### 示例

基础示例：

```php
<?php

use Enqueue\Dsn\Dsn;

$dsn = Dsn::parseFirst('mysql+pdo://user:password@localhost:3306/database?connection_timeout=123');

$dsn->getSchemeProtocol(); // 'mysql'
$dsn->getScheme(); // 'mysql+pdo'
$dsn->getSchemeExtensions(); // ['pdo']
$dsn->getUser(); // 'user'
$dsn->getPassword(); // 'password'
$dsn->getHost(); // 'localhost'
$dsn->getPort(); // 3306

$dsn->getQueryString(); // 'connection_timeout=123'
$dsn->getQuery(); // ['connection_timeout' => '123']
$dsn->getString('connection_timeout'); // '123'
$dsn->getDecimal('connection_timeout'); // 123
```

Parse Cluster DSN：

```php
<?php

use Enqueue\Dsn\Dsn;

$dsns = Dsn::parse('mysql+pdo://user:password@foo:3306,bar:5678/database?connection_timeout=123');

count($dsns); // 2

$dsns[0]->getUser(); // 'user'
$dsns[0]->getPassword(); // 'password'
$dsns[0]->getHost(); // 'foo'
$dsns[0]->getPort(); // 3306

$dsns[1]->getUser(); // 'user'
$dsns[1]->getPassword(); // 'password'
$dsns[1]->getHost(); // 'bar'
$dsns[1]->getPort(); // 5678
```

有些部分可以省略：

```php
<?php
use Enqueue\Dsn\Dsn;

$dsn = Dsn::parseFirst('sqs:?key=aKey&secret=aSecret&token=aToken');

$dsn->getSchemeProtocol(); // 'sqs'
$dsn->getScheme(); // 'sqs'
$dsn->getSchemeExtensions(); // []
$dsn->getUser(); // null
$dsn->getPassword(); // null
$dsn->getHost(); // null
$dsn->getPort(); // null

$dsn->getString('key'); // 'aKey'
$dsn->getString('secret'); // 'aSecret'
```

获取输入的查询参数：

```php
<?php
use Enqueue\Dsn\Dsn;

$dsn = Dsn::parseFirst('sqs:?decimal=12&octal=0666&float=1.2&bool=1&array[0]=val&array[1]=123');

$dsn->getDecimal('decimal'); // 12
$dsn->getOctal('decimal'); // 0666
$dsn->getFloat('float'); // 1.2
$dsn->getBool('bool'); // true
$dsn->getArray('array')->getString(0); // val
$dsn->getArray('array')->getDecimal(1); // 123
$dsn->getArray('array')->toArray(); // [val]
```

如果 DSN 无效，则抛出异常：

```php
<?php
use Enqueue\Dsn\Dsn;

$dsn = Dsn::parseFirst('foo'); // 在这里抛出异常
```

如果无法转换查询参数，则抛出异常：

```php
<?php
use Enqueue\Dsn\Dsn;

$dsn = Dsn::parseFirst('mysql:?connection_timeout=notInt');

$dsn->getDecimal('connection_timeout'); // 在这里抛出异常
```

[返回目录](index.md)