# onClose
## Description:
```php
callback Connection::$onClose
```

Ce rappel fonctionne de la même manière que le rappel [Worker::$onClose](../worker/on-close.md), à la différence qu'il est uniquement valable pour la connexion actuelle, c'est-à-dire qu'il permet de définir un rappel onClose pour une connexion spécifique.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Lorsqu'un événement de connexion se produit
$worker->onConnect = function(TcpConnection $connection)
{
    // Définir le rappel onClose de la connexion
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connexion fermée\n";
    };
};
// Exécuter le worker
Worker::runAll();
```

Le code ci-dessus est équivalent à l'exemple suivant

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définir le rappel onClose pour toutes les connexions
$worker->onClose = function(TcpConnection $connection)
{
    echo "connexion fermée\n";
};
// Exécuter le worker
Worker::runAll();
```
