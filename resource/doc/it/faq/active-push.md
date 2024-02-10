# Come inviare attivamente i messaggi

1. È possibile programmare l'invio periodico dei dati utilizzando un timer
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Dopo l'avvio del processo, invia periodicamente i dati al client
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Notificare a Workerman di inviare i dati quando si verifica un determinato evento in un altro progetto
Vedere [Domande frequenti - Invio in un altro progetto](push-in-other-project.md)
