# destroy
## Erklärung:
```php
void Connection::destroy()
```
Schließt die Verbindung sofort.

Im Gegensatz zu "close" wird die Verbindung nach dem Aufruf von "destroy" sofort geschlossen, auch wenn der Sendepuffer dieser Verbindung noch Daten enthält, die noch nicht an die Gegenstelle gesendet wurden. Die Verbindung wird sofort geschlossen und sofort das "onClose" Rückrufereignis dieser Verbindung ausgelöst.

## Parameter

Keine Parameter

## Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// Starte den Worker
Worker::runAll();
```
