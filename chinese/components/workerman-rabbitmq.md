# workerman/rabbitmq

workerman/rabbitmq是一个异步RabbitMQ客户端，使用AMQP协议。

## 项目地址：
https://github.com/walkor/rabbitmq

## 安装：
```
composer require workerman/rabbitmq
```

## 示例


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
        echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";
        $channel->consume(
            function (Message $message, Channel $channel, Client $client) {
                echo " [x] Received ", $message->content, "\n";
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
        echo " [x] Sending 'Hello World!'\n";
        return $channel->publish('Hello World!', [], '', 'hello')->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo " [x] Sent 'Hello World!'\n";
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


