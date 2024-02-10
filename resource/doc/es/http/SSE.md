# SSE
**Esta característica requiere workerman>=4.0.0**

SSE, también conocido como Eventos enviados por el servidor (Server-sent Events), es una técnica de envío de servidor. Su esencia es que, después de que el cliente envía una solicitud HTTP con la cabecera `Accept: text/event-stream`, la conexión no se cierra, y el servidor puede seguir enviando datos al cliente a través de esta conexión.

La diferencia entre SSE y WebSocket es:
* SSE solo admite el envío del servidor al cliente; WebSocket permite la comunicación bidireccional.
* SSE admite reconexión automática por defecto; WebSocket requiere ser implementada manualmente.
* SSE solo puede transmitir texto UTF-8, los datos binarios deben ser codificados a UTF-8 antes de ser transmitidos; WebSocket admite la transmisión de datos UTF-8 y binarios por defecto.
* SSE incluye tipos de mensajes incorporados; WebSocket requiere ser implementada manualmente.

### Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Si la cabecera Accept es text/event-stream, es una solicitud SSE
    if ($request->header('accept') === 'text/event-stream') {
        // Envía una respuesta con la cabecera Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Envía datos al cliente periódicamente
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Se elimina el temporizador cuando se cierra la conexión para evitar fugas de memoria
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Envía el evento 'message' con los datos 'hello', el ID del mensaje puede no ser especificado
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Ejecutar el worker
Worker::runAll();
```

Código JavaScript del cliente
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Imprime hello
}, false);
```
