# getRemotePort
## Description:
```php
int Connection::getRemotePort()
```
Obtenez le port du client pour cette connexion.

## Parameters
No parameters

## Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "Nouvelle connexion depuis l'adresse " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Exécutez le worker
Worker::runAll();
```
