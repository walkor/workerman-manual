# Cómo enviar mensajes activamente

1. Se puede programar un temporizador para enviar datos periódicamente
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// En el inicio del proceso, enviar datos periódicamente a los clientes
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Notificar a Workerman para enviar datos cuando ocurra un evento en otro proyecto
Consulte [Preguntas frecuentes - Envío en otro proyecto](push-in-other-project.md)
