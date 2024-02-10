```php
void Connection::close(mixed $data = '')
```

Sicher schließen Sie die Verbindung und lösen Sie das ```onClose``` Rückruf der Verbindung aus.

Obwohl UDP keine Verbindung hat, wird das entsprechende AsyncUdpConnection-Objekt jedoch die ganze Zeit im Speicher beibehalten. Sie müssen die close-Methode aufrufen, um das entsprechende UDP-Verbindungsobjekt freizugeben. Andernfalls bleibt dieses UDP-Verbindungsobjekt im Speicher bestehen und führt zu Speicherlecks.

## Parameter

 ``` $data ```

Optionales Argument, um die zu sendenden Daten anzugeben (wenn ein Protokoll angegeben ist, wird automatisch die encode-Methode des Protokolls aufgerufen, um die ```$data```-Daten zu verpacken), nachdem die Daten gesendet wurden, wird die Verbindung geschlossen, und anschließend wird der onClose-Rückruf ausgelöst.

Die Daten dürfen nicht mehr als 65507 Byte groß sein, sonst erfolgt der Versand fehl.

### Beispiel 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 Sekunde später starten Sie einen UDP-Client, verbinden sich mit dem Port 1234 und senden die Zeichenfolge "hi".
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Empfangene Daten vom Server zurück "hello"
            echo "recv $data\r\n";
            // Verbindung schließen
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Daten vom AsyncUdpConnection-Client empfangen, die Zeichenfolge "hello" zurückgeben
    $connection->send("hello");
};
Worker::runAll();             
```

Nach Ausführung wird etwas Ähnliches gedruckt:
```
recv hello
```
