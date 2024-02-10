# How to actively push messages

1. You can use a timer to push data at regular intervals
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Push data to clients at regular intervals after the process starts
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Notify Workerman to push data when an event occurs in another project
See also: [FAQ-Push in other projects](push-in-other-project.md)
