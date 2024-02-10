## Schließen
### Erklärung:
```php
void Connection::close(mixed $data = '')
```

Sicheres Schließen der Verbindung.

Das Aufrufen von `close` wartet darauf, dass die Daten im Sendepuffer gesendet werden, bevor die Verbindung geschlossen wird, und löst das `onClose`-Rückrufereignis der Verbindung aus.

### Parameter
``` $data ```

Optionaler Parameter, um zu sendende Daten (wenn ein Protokoll angegeben ist, wird die `encode`-Methode des Protokolls automatisch aufgerufen, um die Daten von `$data` zu verpacken). Nachdem die Daten gesendet wurden, wird die Verbindung geschlossen und anschließend wird das `onClose`-Ereignis ausgelöst.

### Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Worker ausführen
Worker::runAll();
```
