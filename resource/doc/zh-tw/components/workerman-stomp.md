# workerman/stomp

STOMP是一個通訊協議。它支援大多數消息隊列，如RabbitMQ、Apollo等。

## 專案地址：
https://github.com/walkor/stomp

## 安裝：
```composer require workerman/stomp```

## 範例
```php
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
       // 訂閱
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // 發布
        $client->send('/topic/foo', 'Hello Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
