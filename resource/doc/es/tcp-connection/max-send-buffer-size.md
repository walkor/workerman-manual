# maxSendBufferSize
## Descripción:
```php
int Connection::$maxSendBufferSize
```

Cada conexión tiene un búfer de envío de capa de aplicación único. Si la velocidad de recepción del cliente es menor que la velocidad de envío del servidor, los datos se almacenarán temporalmente en el búfer de la capa de aplicación y esperarán a ser enviados.

Esta propiedad se utiliza para establecer el tamaño del búfer de envío de la capa de aplicación de la conexión actual. Si no se establece, el valor predeterminado es [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1MB).

Esta propiedad afecta a la devolución de llamada [onBufferFull](../worker/on-buffer-full.md).

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Establecer el tamaño del búfer de envío de capa de aplicación de la conexión actual en 102400 bytes
    $connection->maxSendBufferSize = 102400;
};
// Ejecutar el worker
Worker::runAll();
```
