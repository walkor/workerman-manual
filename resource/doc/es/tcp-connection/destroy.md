# destroy
## Descripción:
```php
void Connection::destroy()
```

Cierra la conexión inmediatamente.

A diferencia de close, después de llamar a destroy, la conexión se cerrará de inmediato incluso si el búfer de envío de la conexión aún tiene datos sin enviar al otro extremo, y se activará de inmediato el callback ```onClose``` de esa conexión.

## Parámetros

Sin parámetros


## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// ejecutar el worker
Worker::runAll();
```
