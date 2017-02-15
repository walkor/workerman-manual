# redis-react
**``` (要求Workerman版本>=3.3.6) ```**

## 安装：
```
composer require clue/redis-react
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Clue\React\Redis\Factory;
use Clue\React\Redis\Client;
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

// 进程启动时
$worker->onWorkerStart = function() {
    global $factory;
    $loop    = Worker::getEventLoop();
    $factory = new Factory($loop);
};

$worker->onMessage = function($connection, $data) {
    global $factory;
    $factory->createClient('localhost:6379')->then(function (Client $client) use ($connection) {
        $client->set('greeting', 'Hello world');
        $client->append('greeting', '!');

        $client->get('greeting')->then(function ($greeting) use ($connection){
            // Hello world!
            echo $greeting . PHP_EOL;
            $connection->send($greeting);
        });

        $client->incr('invocation')->then(function ($n) use ($connection){
            echo 'This is invocation #' . $n . PHP_EOL;
            $connection->send($n);
        });
    });
};

Worker::runAll();
```

## 文档：
https://github.com/clue/php-redis-react

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值



