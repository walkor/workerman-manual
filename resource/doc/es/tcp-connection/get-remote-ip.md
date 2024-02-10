# getRemoteIp
## Descripción:
```php
string Connection::getRemoteIp()
```
Obtiene la dirección IP del cliente de esta conexión.

## Parámetros
Sin parámetros

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nueva conexión desde la dirección IP " . $connection->getRemoteIp() . "\n";
};
// Ejecutar el worker
Worker::runAll();
```
