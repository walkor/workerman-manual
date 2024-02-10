# Die reConnect Methode
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Workerman-Version >= 3.3.5 erforderlich) ```

Die Methode zum erneuten Verbinden. Wird normalerweise im ```onClose```-Rückruf aufgerufen, um die Wiederherstellung der Verbindung nach einem Verbindungsabbruch zu realisieren.

Bei Netzwerkproblemen oder dem Neustart des Serverdienstes des Gegenübers kann durch den Aufruf dieser Methode eine erneute Verbindung hergestellt werden.


### Parameter
 ``` $delay ```

Zeitverzögerung bis zur erneuten Verbindungsherstellung. Die Einheit ist Sekunden, unterstützt Dezimalstellen und kann bis auf Millisekunden genau sein.

Wenn kein Wert übergeben wird oder der Wert 0 beträgt, steht dies für eine sofortige erneute Verbindungsherstellung.

Es ist ratsam, den Parameter zu übergeben, um die Verzögerung der erneuten Verbindung zu ermöglichen, um zu vermeiden, dass ein hoher CPU-Verbrauch aufgrund einer dauerhaft nicht verfügbaren Verbindung beim Gegenüber entsteht.


### Rückgabewert
Kein Rückgabewert

### Beispiel

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "Nachricht empfangen: $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Bei Verbindungsabbruch, in 1 Sekunde erneut verbinden
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Anmerkung**
> Nach erfolgreicher erneuter Verbindungsherstellung wird die onConnect-Methode von $con erneut aufgerufen (sofern vorhanden). Manchmal möchten wir jedoch, dass die onConnect-Methode nur einmal ausgeführt wird und bei der erneuten Verbindung nicht erneut aufgerufen wird. Ein Beispiel dazu finden Sie unten:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "Nachricht empfangen: $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Bei Verbindungsabbruch, in 1 Sekunde erneut verbinden
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
