# onBufferDrain
## Descripción:
```php
callback Worker::$onBufferDrain
```

Cada conexión tiene un búfer de envío de capa de aplicación, cuyo tamaño está determinado por ```TcpConnection::$maxSendBufferSize```, con un valor predeterminado de 1 MB, que puede ser modificado manualmente para cambiar el tamaño, y este cambio afectará a todas las conexiones.

Esta devolución de llamada se activa después de que se hayan enviado todos los datos del búfer de envío de la capa de aplicación. Por lo general, se utiliza en combinación con onBufferFull, por ejemplo, deteniendo el envío de datos al extremo opuesto en onBufferFull y reanudando la escritura de datos en onBufferDrain.

## Parámetros de la función de devolución de llamada
 ``` $connection ```

El objeto de la conexión, es decir, la [instancia de TcpConnection](../tcp-connection.md), se utiliza para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md), [cerrar la conexión](../tcp-connection/close.md), etc.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// Run worker
Worker::runAll();
```

Nota: Aparte de usar una función anónima como devolución de llamada, también se puede [consultar aquí](../faq/callback_methods.md) para utilizar otras formas de devolución de llamada.

## Ver también
onBufferFull: Se activa cuando el búfer de envío de capa de aplicación de la conexión está lleno
