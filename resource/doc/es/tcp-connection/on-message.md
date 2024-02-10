# onMessage
## Descripción:
```php
callback Connection::$onMessage
```

Tiene el mismo efecto que el callback [Worker::$onMessage](../worker/on-message.md), pero la diferencia es que solo es válido para la conexión actual, es decir, se puede establecer el callback onMessage para una conexión específica.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Cuando se produce un evento de conexión de cliente
$worker->onConnect = function(TcpConnection $connection)
{
    // Establecer el callback onMessage para la conexión
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('recepción exitosa');
    };
};
// Ejecutar el worker
Worker::runAll();
```

El código anterior es equivalente al siguiente:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establecer directamente el callback onMessage para todas las conexiones
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('recepción exitosa');
};
// Ejecutar el worker
Worker::runAll();
```
