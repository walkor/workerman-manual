# onMessage
## Descripción:
```php
callback Worker::$onMessage
```

Se dispara cuando el cliente envía datos a través de la conexión (cuando Workerman recibe datos).

## Parámetros de la función de devolución de llamada

 ``` $connection ```

Objeto de conexión, es decir, una instancia de [TcpConnection](../tcp-connection.md), utilizado para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md), [cerrar la conexión](../tcp-connection/close.md), etc.

 ``` $data ```

Los datos enviados por el cliente a través de la conexión, si el Worker ha especificado un protocolo, entonces $data es el resultado de decodificar según el protocolo correspondiente. El tipo de datos depende de la implementación de `decode()` del protocolo, siendo `websocket`, `text` y `frame` de tipo string, y el protocolo HTTP de tipo [`Workerman\Protocols\Http\Request`](../http/request.md).

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('recepción satisfactoria');
};
// Ejecutar el worker
Worker::runAll();
```

Nota: Además de usar una función anónima como devolución de llamada, también se puede [consultar aquí](../faq/callback_methods.md) para ver otras formas de escribir devoluciones de llamada.
