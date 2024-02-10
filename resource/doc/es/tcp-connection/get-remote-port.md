# getRemotePort
## Descripción:
```php
int Connection::getRemotePort()
```

Obtiene el puerto del cliente de esta conexión.

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
    echo "Nueva conexión desde la dirección " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Ejecutar el worker
Worker::runAll();
```
