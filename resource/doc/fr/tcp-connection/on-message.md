# onMessage
## Description:
```php
callback Connection::$onMessage
```


Cette fonction est similaire au rappel [Worker::$onMessage](../worker/on-message.md), mais elle est valable uniquement pour la connexion actuelle, c'est-à-dire qu'elle peut être définie pour un rappel onMessage spécifique à une connexion.


## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Lorsqu'un événement de connexion client se produit
$worker->onConnect = function(TcpConnection $connection)
{
    // Définir le rappel onMessage pour la connexion
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('réception réussie');
    };
};
// Exécuter le worker
Worker::runAll();
```


Le code ci-dessus a le même effet que celui ci-dessous

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définir directement le rappel onMessage pour toutes les connexions
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('réception réussie');
};
// Exécuter le worker
Worker::runAll();
```
