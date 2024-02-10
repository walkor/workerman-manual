# connect Methode

```php
void AsyncUdpConnection::connect()
```

Führt eine asynchrone Verbindung durch. Diese Methode kehrt sofort zurück.

### Parameter
Keine Parameter

### Rückgabewert
Kein Rückgabewert

### Beispiel

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Startet einen UDP-Client nach 1 Sekunde, verbindet sich mit dem Port 1234 und sendet den String "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Empfängt Daten "hello" vom Server
            echo "recv $data\r\n";
            // Verbindung schließen
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Empfängt Daten vom AsyncUdpConnection-Client und sendet den String "hello" zurück
    $connection->send("hello");
};
Worker::runAll();             
```

Nach der Ausführung wird ungefähr Folgendes gedruckt:
```
recv hello
```
