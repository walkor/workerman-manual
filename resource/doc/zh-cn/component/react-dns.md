# react/dns
**``` (要求Workerman版本>=3.3.6) ```**
## 安装：
```
composer require react/dns
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global   $dns;
    $loop    = Worker::getEventLoop();
    $factory = new React\Dns\Resolver\Factory();
    $dns     = $factory->create('8.8.8.8', $loop);
};

$worker->onMessage = function($connection, $host) {
    global $dns;
    $dns->resolve($host)->then(function($ip) use($host, $connection) {
        $connection->send("$host: $ip");
    },function($e) use($host, $connection){
        $connection->send("$host: {$e->getMessage()}");
    });
};

Worker::runAll();
```

## 文档：
https://github.com/reactphp/dns

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值


