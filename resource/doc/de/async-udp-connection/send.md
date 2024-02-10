# Die Methode send

```php
void AsyncUdpConnection::send(string $data)
```

Führt eine asynchrone Verbindung operation aus. Diese Methode gibt sofort zurück.

### Parameter
``` $data ```
Die Daten, die an den Server gesendet werden sollen. Die Daten dürfen nicht mehr als 65507 Bytes betragen (Die maximale Übertragungsgröße eines UDP-Datenpakets beträgt 65507 Bytes), da sonst der Versand fehlschlägt.

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
    // Startet einen UDP-Client nach 1 Sekunde, stellt eine Verbindung mit dem Port 1234 her und sendet den String "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Empfangene Daten vom Server: "hello"
            echo "recv $data\r\n";
            // Verbindung schließen
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Daten vom AsyncUdpConnection-Client empfangen, String "hello" zurückschicken
    $connection->send("hello");
};
Worker::runAll();
```

Nach der Ausführung wird "recv hello" gedruckt.
