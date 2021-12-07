There is an interprocess/multi-server communication component for workerman named channel (https://github.com/walkor/Channel).

Install workerman\channel with command
  composer require workerman/channel
or if you just downloaded composer.phar without install
  php composer.phar require workerman/channel
this will also install workerman if it`s not installed

Example.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Channel Server. 
$channel_server = new Channel\Server('0.0.0.0', 2206);

$worker = new Worker('websocket://0.0.0.0:4236');
// 6 processes.
$worker->count=6;
$worker->onWorkerStart = function($worker)
{
    // Channel client connect to Channel Server.
    Channel\Client::connect('127.0.0.1', 2206);
    // Subscribe broadcast event ('broadcast' can be any name)
    Channel\Client::on('broadcast', function($data)use($worker){
        foreach ($worker->connections as $connection) {
           $connection->send($data);
        }
    });

    // and you can even subscribe any worker by it`s id.
    Channel\Client::on($worker->id, function($data)use($worker){
        foreach ($worker->connections as $connection) {
           $connection->send($data);
        }
        // or send data to the certain connection by connection id.
        $connection[1]->send($data);
    });
};

$worker->onMessage = function($connection, $data) {
    // Publish broadcast event to all worker processes.
    Channel\Client::publish('broadcast', $data);
    // Or you can send data to the certain worker by id.
    $target_worker_id = 0;
    Channel\Client::publish($target_worker_id, $data);
};

Worker::runAll();
```
