# Workerman/Stomp

STOMP هو بروتوكول اتصال. إنه يدعم معظم طوابير الرسائل مثل RabbitMQ وApollo وغيرها.

## عنوان المشروع:
https://github.com/walkor/stomp

## التثبيت:
```composer require workerman/stomp```

## مثال
```php
<?php
استخدام Workerman\Worker;
استخدام Workerman\Timer;
استخدام Workerman\Stomp\Client;
استخدام Workerman\RedisQueue\Client;
تتطلب_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $client = new Workerman\Stomp\Client('stomp://127.0.0.1:61613');
    $client->onConnect = function(Client $client) {
       // الاشتراك
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // النشر
        $client->send('/topic/foo', 'Hello Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
