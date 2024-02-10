# worker
## Description:
```php
Worker Connection::$worker
```

Cette propriété est en lecture seule, c'est-à-dire l'instance de worker à laquelle l'objet connection actuel appartient.


## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Lorsqu'un client envoie des données, les transmettre à tous les autres clients maintenus par le processus actuel
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Exécuter le worker
Worker::runAll();
```
