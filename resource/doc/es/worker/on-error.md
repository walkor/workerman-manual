# onError
## Descripción:
```php
callback Worker::$onError
```

Se activa cuando ocurre un error en la conexión del cliente.

Los tipos de error actuales son:

1. Error al llamar a Connection::send debido a una desconexión del cliente (seguido de la activación de la devolución de llamada onClose) ```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. Después de activar onBufferFull (el búfer de envío está lleno), sigue llamando a Connection::send y el búfer de envío sigue lleno, lo que provoca un fallo en el envío (no se activará la devolución de llamada onClose)```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. Fallo en la conexión asincrónica con AsyncTcpConnection (seguido de la activación de la devolución de llamada onClose) ```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client returned error message)```

## Parámetros de la función de devolución de llamada

 ``` $connection ```

Objeto de conexión, es decir, la instancia de [TcpConnection](../tcp-connection.md), utilizada para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md), [cerrar la conexión](../tcp-connection/close.md), etc.

 ``` $code ```

Código de error

 ``` $msg ```

Mensaje de error

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// Ejecutar el worker
Worker::runAll();
```

Consejo: además de usar una función anónima como devolución de llamada, también se puede [consultar aquí](../faq/callback_methods.md) para utilizar otros métodos de devolución de llamada.
