# SSE
**Questa funzione richiede workerman>=4.0.0**

SSE, ovvero Server-sent Events, è una tecnologia di push dal server. Essenzialmente, quando il client invia una richiesta http con l'intestazione `Accept: text/event-stream`, la connessione rimane aperta e il server può continuare a inviare dati al client su questa connessione.

La differenza con il websocket è la seguente:
*   SSE consente solo al server di inviare dati al client; il WebSocket consente la comunicazione bidirezionale.
*   SSE supporta il riconoscimento della disconnessione come impostazione predefinita; il WebSocket richiede un'implementazione personalizzata.
*   SSE può trasmettere solo testo UTF-8, i dati binari devono essere codificati in UTF-8 prima di essere trasmessi; il WebSocket supporta per impostazione predefinita la trasmissione di dati UTF-8 e binari.
*   SSE ha un tipo di messaggio predefinito; il WebSocket richiede un'implementazione personalizzata.

### Esempio
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
    // Se l'intestazione Accept è text/event-stream, si tratta di una richiesta SSE.
    if ($request->header('accept') === 'text/event-stream') {
        // Inviare prima una risposta con intestazione Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Inviare dati al client con cadenza temporale
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Eliminare il timer quando la connessione è chiusa per evitare accumuli di timer e perdite di memoria
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Inviare l'evento 'message' con i dati 'hello', l'ID del messaggio può essere omesso
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Avviare il worker
Worker::runAll();
```

Codice JavaScript del client
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Output: hello
}, false);
```
