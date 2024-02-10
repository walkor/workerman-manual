# onMessage
## Erklärung:
```php
Rückruffunktion Worker::$onMessage
```

Wird aufgerufen, wenn der Client Daten über die Verbindung sendet (Workerman Daten empfängt).

## Parameter der Rückruffunktion

 ``` $connection ```

Verbindung-Objekt, das heißt [TcpConnection Instanz](../tcp-connection.md), um die Client-Verbindung zu steuern, wie [Daten senden](../tcp-connection/send.md), [Verbindung schließen](../tcp-connection/close.md), usw.

 ``` $data ```

Die vom Client übertragenden Daten. Wenn der Worker ein Protokoll spezifiziert hat, dann ist $data das decodierte Daten entsprechend des Protokolls. Der Datentyp hängt von der `decode()` Implementierung des Protokolls ab. `websocket`, `text`, `frame` sind Zeichenketten, das HTTP-Protokoll ist ein [`Workerman\Protocols\Http\Request`](../http/request.md) Objekt.

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('Empfang erfolgreich');
};
// Worker ausführen
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktion als Rückruffunktion kann auch [hier](../faq/callback_methods.md) auf andere Rückrufschreibweisen verwiesen werden.
