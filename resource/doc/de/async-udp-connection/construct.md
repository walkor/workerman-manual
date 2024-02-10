# __construct Methode
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Erstellt ein UDP-Verbindungsobjekt.

AsyncUdpConnection ermöglicht es Workerman, als Client UDP-Daten mit einem Remote-Server zu übertragen.

## Parameter
Parameter: ``` remote_address ```

Die Adresse der Verbindung, z.B.
 ``` udp://192.168.1.1:1234 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```

## Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Startet einen UDP-Client nach 1 Sekunde, stellt eine Verbindung mit dem Port 1234 her und sendet die Zeichenkette "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Empfangene Daten vom Server "hello"
            echo "recv $data\r\n";
            // Verbindung schließen
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Empfängt Daten vom AsyncUdpConnection-Client und sendet die Zeichenkette "hello" zurück
    $connection->send("hello");
};
Worker::runAll();    
```

Nach der Ausführung wird etwas Ähnliches gedruckt:
``` 
recv hello
```
