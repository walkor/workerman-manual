# Comment pousser activement des messages

1. Vous pouvez utiliser un minuteur pour envoyer des données de manière périodique :

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Après le démarrage du processus, envoyez périodiquement des données aux clients
$worker->onWorkerStart = function($worker){
    Timer::add(1, function() use ($worker) {
        foreach ($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Lorsqu'un événement spécifique se produit dans un autre projet, notifiez Workerman pour envoyer des données. Voir [Questions fréquemment posées - Envoi à partir d'un autre projet](push-in-other-project.md)
