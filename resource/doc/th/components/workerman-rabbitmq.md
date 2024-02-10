workerman/rabbitmq เป็นไคลเอ็นต์ RabbitMQ แบบไม่ซ้ำซ้อนที่ใช้ AMQP โปรโตคอล

## ที่อยู่โครงการ:
https://github.com/walkor/rabbitmq

## การติดตั้ง:
```composer require workerman/rabbitmq```

## ตัวอย่าง

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
        echo ' [*] กำลังรอข้อความ หากต้องการออกให้กด CTRL+C', "\n";
        $channel->consume(
            function (Message $message, Channel $channel, Client $client) {
                echo " [x] ได้รับ ", $message->content, "\n";
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
        echo " [x] กำลังส่ง 'สวัสดีชาวโลก!'\n";
        return $channel->publish('สวัสดีชาวโลก!', [], '', 'hello')->then(function () use ($channel) {
            return $channel;
        });
    })->then(function (Channel $channel) {
        echo " [x] ส่ง 'สวัสดีชาวโลก!'\n";
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
