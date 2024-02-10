# workerman/rabbitmq

workerman/rabbitmq은 AMQP 프로토콜을 사용하는 비동기 RabbitMQ 클라이언트입니다.

## 프로젝트 주소:
https://github.com/walkor/rabbitmq

## 설치:
```composer require workerman/rabbitmq```

## 예제


**receive.php**

```php
<?php
use Bunny\Channel;
use Bunny\Message;
use Workerman\Worker;
use Workerman\RabbitMQ\Client;

require __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function() {
    (new Client())->connect()->then(function (Client $client) {
        return $client->channel();
    })->then(function (Channel $channel) {
        return $channel->queueDeclare('hello', false, false, false, false)->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo ' [*] 메시지를 기다리는 중입니다. 종료하려면 CTRL+C를 누르세요', "\n";
        $channel->consume(
            function (Message $message, Channel $channel, Client $client) {
                echo " [x] 받은 메시지: ", $message->content, "\n";
            },
            'hello',
            '',
            false,
            true
        );
    });
};
Worker::runAll();
```

**send.php**
```php
<?php
use Bunny\Channel;
use Bunny\Message;
use Workerman\Worker;
use Workerman\RabbitMQ\Client;

require __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function() {
    (new Client())->connect()->then(function (Client $client) {
        return $client->channel();
    })->then(function (Channel $channel) {
        return $channel->queueDeclare('hello', false, false, false, false)->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo " [x] 'Hello World!' 전송 중\n";
        return $channel->publish('Hello World!', [], '', 'hello')->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo " [x] 'Hello World!' 전송 완료\n";
        $client = $channel->getClient();
        return $channel->close()->then(function () use ($client) {
            return $client;
        });
    })->then(function (Client $client) {
        $client->disconnect();
    });
};
Worker::runAll();
```
