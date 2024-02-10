# onBufferFull
## Descripción:
```php
callback Worker::$onBufferFull
```
Cada conexión tiene un búfer de envío de capa de aplicación separado. Si la velocidad de recepción del cliente es menor que la velocidad de envío del servidor, los datos se almacenarán en el búfer de la capa de aplicación. Si el búfer está lleno, se activará el callback onBufferFull.

El tamaño del búfer es [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), con un valor predeterminado de 1 MB. Se puede configurar dinámicamente el tamaño del búfer de la conexión actual de la siguiente manera:
```php
// Establecer el tamaño del búfer de envío de la conexión actual, en bytes
$connection->maxSendBufferSize = 102400;
```
También se puede utilizar [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) para establecer el tamaño predeterminado del búfer de todas las conexiones, como en el siguiente ejemplo:
```php
use Workerman\Connection\TcpConnection;
// Establecer el tamaño predeterminado del búfer de envío de la capa de aplicación para todas las conexiones, en bytes
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

Este callback **puede** activarse inmediatamente después de llamar a Connection::send, por ejemplo, al enviar grandes cantidades de datos o al enviar datos continuamente rápida a la parte opuesta. Esto se debe a que, por razones de red u otras, los datos se acumulan en el búfer de envío correspondiente a la conexión, lo que provoca que se active cuando se supera el límite de ```TcpConnection::$maxSendBufferSize```.

Cuando ocurre el evento onBufferFull, normalmente los desarrolladores deben tomar medidas, como detener el envío de datos a la parte opuesta y esperar a que se envíen todos los datos en el búfer de envío (evento onBufferDrain), entre otras.

Cuando se llama a Connection::send(`$A`) y se desencadena onBufferFull, sin importar cuánto sea `$A`, incluso si supera ```TcpConnection::$maxSendBufferSize```, los datos a enviar en este momento todavía se colocarán en el búfer de envío. Esto significa que la cantidad de datos colocados en el búfer de envío puede ser mucho mayor que ```TcpConnection::$maxSendBufferSize```. Cuando los datos en el búfer de envío superan ```TcpConnection::$maxSendBufferSize```, si se continúa llamando a Connection::send(`$B`), los datos de `$B` no se colocarán en el búfer de envío, sino que se descartarán, desencadenando el callback `onError`.

En resumen, mientras el búfer de envío no esté lleno, incluso si solo hay espacio para un byte, llamar a Connection::send(```$A```) definitivamente colocará ```$A``` en el búfer de envío. Si después de colocar los datos en el búfer de envío, su tamaño supera el límite de ```TcpConnection::$maxSendBufferSize```, se activará el callback onBufferFull.

## Parámetros de la función de devolución de llamada

``` $connection ```

El objeto de conexión, es decir, una instancia de [TcpConnection](../tcp-connection.md), que se usa para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md), [cerrar la conexión](../tcp-connection/close.md), entre otros.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull y no enviar de nuevo\n";
};
// Ejecutar el worker
Worker::runAll();
```

Nota: Además de utilizar una función anónima como devolución de llamada, también puede consultar [aquí](../faq/callback_methods.md) para ver otras formas de devolución de llamada.

## Ver también
onBufferDrain: Se activa cuando se han enviado todos los datos del búfer de envío de la capa de la aplicación de una conexión.
