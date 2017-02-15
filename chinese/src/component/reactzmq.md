# react/zmq
**``` (要求Workerman版本>=3.3.6) ```**

## 安装：
```
composer require react/zmq
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global   $pull;
    $loop    = Worker::getEventLoop();
    $context = new React\ZMQ\Context($loop);
    $pull    = $context->getSocket(ZMQ::SOCKET_PULL);
    $pull->bind('tcp://127.0.0.1:5555');

    $pull->on('error', function ($e) {
        var_dump($e->getMessage());
    });

    $pull->on('message', function ($msg) {
        echo "Received: $msg\n";
    });
};

Worker::runAll();
```

## 文档：
https://github.com/reactphp/zmq

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值



