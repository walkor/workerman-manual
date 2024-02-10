# close
## Descripción:
```php
void Connection::close(mixed $data = '')
```

Cerrar de forma segura la conexión.

Llamar a close esperará a que se envíen los datos del búfer de envío antes de cerrar la conexión, y activará la devolución de llamada de cierre ```onClose```.

## Parámetros

 ``` $data ```

Parámetro opcional, los datos a enviar (si se especifica un protocolo, automáticamente llamará al método de codificación del protocolo para empaquetar los datos de ```$data```), después de enviar los datos, cerrará la conexión y activará la devolución de llamada de cierre onClose.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Ejecutar el worker
Worker::runAll();
```
