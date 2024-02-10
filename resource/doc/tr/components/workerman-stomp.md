# webman/stomp

STOMP bir iletişim protokolüdür. Çoğu mesaj kuyruğunu desteklemektedir, örneğin RabbitMQ, Apollo vb.

## Proje Adresi:
https://github.com/walkor/stomp

## Kurulum:
```php
composer require workerman/stomp
```

## Örnek
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
       // Abone ol
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // Yayınla
        $client->send('/topic/foo', 'Merhaba Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
