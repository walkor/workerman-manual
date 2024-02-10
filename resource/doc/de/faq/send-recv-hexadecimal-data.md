# Wie man hexadezimale Daten sendet und empfängt

**Empfangen von hexadezimalen Daten**

Nach dem Empfangen der Daten können sie mit der Funktion `bin2hex($data)` in hexadezimale Daten umgewandelt werden.

**Senden von hexadezimalen Daten**

Bevor Daten gesendet werden, können die hexadezimalen Daten mit `hex2bin($data)` in binäre Daten umgewandelt werden.


**Beispiel:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Erhalte hexadezimale Daten
    $hex_data = bin2hex($data);
    // Sende hexadezimale Daten an den Client
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
