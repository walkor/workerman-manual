# workerman/rabbitmq

workerman/rabbitmq, AMQP protokolünü kullanan bir asenkron RabbitMQ istemcisidir.

## Proje Linki:
https://github.com/walkor/rabbitmq

## Kurulum:
```composer require workerman/rabbitmq```

## Örnekler


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
        echo ' [*] Mesajlar için bekleniyor. Çıkmak için CTRL+C tuşlarına basın', "\n";
        $channel->consume(
            function (Message $message, Channel $channel, Client $client) {
                echo " [x] Alındı: ", $message->content, "\n";
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
        echo " [x] 'Hello World!' gönderiliyor\n";
        return $channel->publish('Hello World!', [], '', 'hello')->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo " [x] 'Hello World!' gönderildi\n";
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
