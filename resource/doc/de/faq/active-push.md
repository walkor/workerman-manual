# Wie man aktiv Nachrichten sendet

1. Sie können einen Timer verwenden, um Daten regelmäßig zu senden
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Push data to clients regularly after the process starts
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Bei einem anderen Projekt Meldungen an Workerman senden, wenn ein bestimmtes Ereignis auftritt
Siehe [FAQ-Wie man in einem anderen Projekt sendet](push-in-other-project.md)
