# ID

## Beschreibung:
```php
int Connection::$id
```

Die ID der Verbindung. Dies ist eine automatisch erhöhte Ganzzahl.

Hinweis: Workerman ist mehrprozessig, und jeder Prozess wird eine automatisch erhöhte Verbindungs-ID beibehalten, so dass es zu Duplikate von Verbindungs-IDs zwischen den Prozessen kommen kann.
Wenn Sie eine nicht wiederholende Verbindungs-ID benötigen, können Sie der Verbindung->id nach Bedarf einen neuen Wert zuweisen, z. B. durch das Hinzufügen des Präfix worker->id.

## Siehe auch
[Eigenschaft connections von Worker](../worker/connections.md)


## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Worker ausführen
Worker::runAll();
```
