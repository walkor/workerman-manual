# Schließen Sie unauthentifizierte Verbindungen

**Frage:**

Wie kann man die Verbindung eines Clients automatisch schließen, der innerhalb einer festgelegten Zeit keine Daten gesendet hat, z.B. wenn innerhalb von 30 Sekunden keine Daten empfangen wurden, wird die Verbindung automatisch geschlossen, um unauthentifizierte Verbindungen dazu zu zwingen, sich innerhalb der festgelegten Zeit zu authentifizieren?

**Antwort:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Fügen Sie dem $connection-Objekt vorübergehend ein „auth_timer_id“-Attribut hinzu, um die ID des Timers zu speichern
    // Schließen Sie die Verbindung nach 30 Sekunden automatisch. Der Client muss innerhalb von 30 Sekunden die Authentifizierung senden, um den Timer zu löschen.
    $connection->auth_timer_id = Timer::add(30, function() use ($connection) {
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
        case 'login':
            ...ausgelassen
            // Authentifizierung erfolgreich, löschen des Timers, um zu verhindern, dass die Verbindung geschlossen wird
            Timer::del($connection->auth_timer_id);
            break;
             ... ausgelassen
    }
    ... ausgelassen
}
```
