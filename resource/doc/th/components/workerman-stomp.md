# Workerman/STOMP

STOMP เป็นโปรโตคอลสำหรับการสื่อสาร มันรองรับคิวข้อความส่วนใหญ่ เช่น RabbitMQ, Apollo และอื่น ๆ

## ที่อยู่ของโปรเจค:
https://github.com/walkor/stomp

## การติดตั้ง:
```sh
composer require workerman/stomp
```

## ตัวอย่าง
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
       // รับ subcribe
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // ส่ง
        $client->send('/topic/foo', 'สวัสดี Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
