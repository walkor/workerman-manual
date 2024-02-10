# workerman/stomp

STOMP là một giao thức truyền thông. Nó hỗ trợ hầu hết các hàng đợi tin nhắn như RabbitMQ, Apollo, v.v.

## Địa chỉ dự án:
https://github.com/walkor/stomp

## Cài đặt:
```composer require workerman/stomp```

## Ví dụ
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
       // Đăng ký
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // Đăng
        $client->send('/topic/foo', 'Xin chào Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
