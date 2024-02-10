# workerman/stomp

STOMPは通信プロトコルです。RabbitMQ、Apolloなどの多くのメッセージキューをサポートしています。

## プロジェクトのアドレス：
https://github.com/walkor/stomp

## インストール：
```bash
composer require workerman/stomp
```

## サンプル
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Stomp\Client;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function() {
    $client = new Workerman\Stomp\Client('stomp://127.0.0.1:61613');
    $client->onConnect = function(Client $client) {
        // Subscribe
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // Publish
        $client->send('/topic/foo', 'Hello Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
