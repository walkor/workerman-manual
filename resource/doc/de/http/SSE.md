# SSE 
**Diese Funktion erfordert workerman>=4.0.0**

SSE, auch bekannt als Server-sent Events, ist eine Technologie für serverseitiges Pushen. Im Wesentlichen sendet der Client eine HTTP-Anfrage mit dem Header `Accept: text/event-stream`, und die Verbindung wird nicht geschlossen. Der Server kann kontinuierlich Daten an den Client über diese Verbindung senden.

Der Unterschied zu Websockets ist folgender:
*   SSE ermöglicht nur serverseitiges Pushen an den Client; Websockets ermöglichen eine bidirektionale Kommunikation.
*   SSE unterstützt standardmäßig das Wiederherstellen der Verbindung bei Verbindungsabbrüchen; WebSockets müssen selbst implementiert werden.
*   SSE kann nur UTF-8-Text übertragen, binäre Daten müssen in UTF-8 codiert und dann übertragen werden; WebSockets unterstützen standardmäßig die Übertragung von UTF-8- und binären Daten.
*   SSE hat eigene Nachrichtentypen; WebSockets müssen selbst implementiert werden.

### Beispiel
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
    // Wenn der Accept-Header "text/event-stream" ist, handelt es sich um eine SSE-Anfrage
    if ($request->header('accept') === 'text/event-stream') {
        // Zuerst eine Antwort mit dem Header "Content-Type: text/event-stream" senden
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Daten periodisch an den Client senden
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Wenn die Verbindung geschlossen ist, muss der Timer gelöscht werden, um einen Speicherleck zu vermeiden
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Die "message"-Ereignis senden, die Daten lauten "hello", die Nachrichten-ID muss nicht übertragen werden
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Worker ausführen
Worker::runAll();
```

Kunden-JavaScript-Code
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Ausgabe: hello
}, false);
```
