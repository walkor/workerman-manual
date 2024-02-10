# defaultMaxSendBufferSize
## Descripción:
```php
static int Connection::$defaultMaxSendBufferSize
```

Esta propiedad es una propiedad estática global que se utiliza para establecer el tamaño predeterminado del búfer de envío de la capa de la aplicación para todas las conexiones. Si no se establece, el valor predeterminado es de ```1MB```. ```Connection::$defaultMaxSendBufferSize``` se puede ajustar dinámicamente, y una vez ajustado, solo será efectivo para las nuevas conexiones que se generen posteriormente.

Esta propiedad afecta a la devolución de llamada [onBufferFull](../worker/on-buffer-full.md).

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Establecer el tamaño predeterminado del búfer de envío de la capa de la aplicación para todas las conexiones
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Establecer el tamaño del búfer de envío de la capa de la aplicación para la conexión actual, sobrescribiendo el valor predeterminado
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Ejecutar el worker
Worker::runAll();
```
