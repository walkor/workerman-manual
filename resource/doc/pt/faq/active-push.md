# Como enviar mensagens de forma ativa

1. Você pode usar um temporizador para enviar dados periodicamente
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Após o início do processo, envie dados periodicamente para o cliente
$worker->onWorkerStart = function($worker){
    Timer::add(1, function() use ($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Notifique o workerman para enviar dados quando ocorrer um determinado evento em outro projeto.
Veja [Perguntas frequentes - Enviando em outros projetos](push-in-other-project.md)
