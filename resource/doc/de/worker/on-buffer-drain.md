# onBufferDrain
## Beschreibung:
```php
callback Worker::$onBufferDrain
```

Jede Verbindung verfügt über einen separaten Anwendungs-Empfangspuffer, dessen Größe durch ```TcpConnection::$maxSendBufferSize``` bestimmt wird. Der Standardwert beträgt 1 MB und kann manuell geändert werden. Diese Änderung wirkt sich auf alle Verbindungen aus.

Dieser Rückruf wird ausgelöst, nachdem alle Daten im Anwendungs-Empfangspuffer erfolgreich gesendet wurden. In der Regel wird es in Verbindung mit `onBufferFull` verwendet, zum Beispiel, um das Senden von Daten an den Peer zu stoppen, wenn `onBufferFull` auftritt, und das Schreiben von Daten in `onBufferDrain` wieder aufzunehmen.

## Parameter der Rückruffunktion
 ``` $connection ```

Verbindungsobjekt, d.h. die Instanz von [TcpConnection](../tcp-connection.md), die für die Bearbeitung der Client-Verbindung, wie das [Senden von Daten](../tcp-connection/send.md), das [Schließen der Verbindung](../tcp-connection/close.md) usw., verwendet wird.

## Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Buffer ist voll und sende nicht erneut\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "Puffer entleeren und fortlaufend senden\n";
};
// Worker ausführen
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruf können auch [andere Rückruf-Schreibweisen hier](../faq/callback_methods.md) verwendet werden.

## Siehe auch
onBufferFull: Wird ausgelöst, wenn der Anwendungs-Empfangspuffer der Verbindung voll ist
