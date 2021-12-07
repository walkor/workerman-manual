# workerman/stomp

STOMP是一个通讯协议。它是支持大多数消息队列如RabbitMQ、Apollo等。

## 项目地址：
https://github.com/walkor/stomp

## 安装：
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
composer require workerman/stomp
```

## 示例
```
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Stomp\Client;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $client = new Workerman\Stomp\Client('stomp://127.0.0.1:61613');
    $client->onConnect = function(Client $client) {
       // 订阅
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // 发布
        $client->send('/topic/foo', 'Hello Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```


