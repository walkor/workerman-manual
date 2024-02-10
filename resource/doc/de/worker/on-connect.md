# onConnect
## Erklärung:
```php
Rückruffunktion Worker::$onConnect
```

Wird aufgerufen, wenn der Client eine Verbindung zu Workerman herstellt (nach Abschluss des TCP-Drei-Wege-Handshakes). Die ```onConnect```-Rückruffunktion wird nur einmal für jede Verbindung ausgelöst.

Hinweis: Das onConnect-Ereignis bedeutet nur, dass der Client den TCP-Drei-Wege-Handshake mit Workerman abgeschlossen hat. Zu diesem Zeitpunkt hat der Client noch keine Daten gesendet. Außer der Verwendung von ```$connection->getRemoteIp()``` gibt es keine Möglichkeit, den Client zu identifizieren oder Informationen über ihn zu erhalten. Daher ist es in einem onConnect-Ereignis nicht möglich festzustellen, wer der Client ist. Um zu wissen, wer der Client ist, muss der Client Authentifizierungsdaten senden, wie z.B. ein bestimmtes Token oder Benutzername und Passwort, und die Authentifizierung im [onMessage-Rückruf](on-message.md) durchführen.

Da UDP eine verbindungslose Protokoll ist, wird beim Verwenden von UDP keine onConnect-Rückruffunktion ausgelöst, und es wird auch kein onClose-Rückruf ausgelöst.

## Parameter der Rückruffunktion

 ``` $connection ```

Verbindungsobjekt, d.h. [TcpConnection-Instanz](../tcp-connection.md), die für die Interaktion mit der Clientverbindung, wie das [Senden von Daten](../tcp-connection/send.md), das [Schließen der Verbindung](../tcp-connection/close.md) usw., verwendet wird


## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "Neue Verbindung von IP " . $connection->getRemoteIp() . "\n";
};
// Worker starten
Worker::runAll();
```

Hinweis: Neben der Verwendung anonymer Funktionen als Rückruf können auch andere Rückrufschreibweisen [hier](../faq/callback_methods.md) verwendet werden.
