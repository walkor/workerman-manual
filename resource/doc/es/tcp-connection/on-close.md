# onClose
## Descripción:
```php
callback Connection::$onClose
```

Esta devolución de llamada tiene el mismo efecto que la devolución de llamada [Worker::$onClose](../worker/on-close.md), la diferencia es que solo es válida para la conexión actual, es decir, se puede establecer una devolución de llamada onClose para una conexión específica.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Cuando ocurre un evento de conexión
$worker->onConnect = function(TcpConnection $connection)
{
    // Establecer la devolución de llamada onClose para la conexión
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "conexión cerrada\n";
    };
};
// Ejecutar el worker
Worker::runAll();
```

El código anterior es equivalente al siguiente

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establecer la devolución de llamada onClose para todas las conexiones
$worker->onClose = function(TcpConnection $connection)
{
    echo "conexión cerrada\n";
};
// Ejecutar el worker
Worker::runAll();
```
