# react/http-client

## 安装：
```
composer require react/http-client
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global   $client;
    $loop    = Worker::getEventLoop();
    $factory = new React\Dns\Resolver\Factory();
    $dns     = $factory->createCached('8.8.8.8', $loop);
    $factory = new React\HttpClient\Factory();
    $client = $factory->create($loop, $dns);
};

$worker->onMessage = function($connection, $host) {
    global     $client;
    $request = $client->request('GET', $host);
    $request->on('error', function(Exception $e) use ($connection) {
        $connection->send($e);
    });
    $request->on('response', function ($response) use ($connection) {
        $response->on('data', function ($data, $response) use ($connection) {
            $connection->send($data);
        });
    });
    $request->end();
};

Worker::runAll();
```

## 文档：
https://github.com/reactphp/http-client

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值



