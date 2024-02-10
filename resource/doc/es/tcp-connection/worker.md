# worker
## Descripción:
```php
Worker Connection::$worker
```

Esta es una propiedad de solo lectura que representa la instancia de worker a la que pertenece el objeto de conexión actual.


## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Cuando un cliente envía datos, se reenvían a todos los demás clientes mantenidos por el proceso actual
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Ejecutar el worker
Worker::runAll();
```
