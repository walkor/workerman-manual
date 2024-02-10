# onError
## Erklärung:
```php
callback Worker::$onError
```

Wird ausgelöst, wenn ein Fehler in der Verbindung des Clients auftritt.

Aktuelle Fehlerarten sind

1. Fehler beim Aufruf von Connection::send aufgrund eines Verbindungsabbruchs des Clients, der zum Fehlschlag führt (anschließend wird der onClose-Callback ausgelöst) ```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. Nach dem Auslösen von onBufferFull (Sendepuffer ist voll) wird dennoch Connection::send aufgerufen und der Sendepuffer ist immer noch voll, was zu einem Sendefehler führt (der onClose-Callback wird nicht ausgelöst)```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. Fehler bei fehlgeschlagener AsyncTcpConnection-Verbindung (anschließend wird der onClose-Callback ausgelöst) ```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client returned error message)```


## Parameter der Rückruffunktion

 ``` $connection ```

Verbindungsobjekt, d. h. [TcpConnection-Instanz](../tcp-connection.md), zum Bearbeiten der Clientverbindung, wie [Daten senden](../tcp-connection/send.md), [Verbindung schließen](../tcp-connection/close.md) usw.

 ``` $code ```

Fehlercode

 ``` $msg ```

Fehlermeldung


## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "Fehler $code $msg\n";
};
// Worker ausführen
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruf können auch [hier](../faq/callback_methods.md) andere Rückrufarten verwendet werden.
