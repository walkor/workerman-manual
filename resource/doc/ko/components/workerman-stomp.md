# workerman/stomp

STOMP은 통신 프로토콜입니다. RabbitMQ, Apollo 등 대부분의 메시지 큐를 지원합니다.

## 프로젝트 주소:
https://github.com/walkor/stomp

## 설치:
```composer require workerman/stomp```

## 예시
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
       // 구독
        $client->subscribe('/topic/foo', function(Client $client, $data) {
            var_export($data);
        });
    };
    $client->onError = function ($e) {
        echo $e;
    };
    Timer::add(1, function () use ($client) {
        // 게시
        $client->send('/topic/foo', 'Hello Workerman STOMP');
    });
    $client->connect();
};
Worker::runAll();
```
