# react/stomp
**``` (要求Workerman版本>=3.3.6) ```**

STOMP是一个通讯协议。它是支持大多数消息队列如RabbitMQ、Apollo等。

## 安装：
```
composer require react/stomp
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
    $factory = new React\Stomp\Factory($loop);
    $client  = $factory->createClient(array('vhost' => '/', 'login' => 'guest', 'passcode' => 'guest'));

    $client
        ->connect()
        ->then(function ($client) use ($loop) {
            $client->subscribe('/topic/foo', function ($frame) {
                echo "Message received: {$frame->body}\n";
            });
        });
};

Worker::runAll();
```

## 文档：
https://github.com/reactphp/stomp

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值



