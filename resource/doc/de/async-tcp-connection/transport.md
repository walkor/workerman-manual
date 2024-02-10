# transport-Eigenschaft
```Erfordert (Workerman >= 3.3.4)```

Setzt die Transporteigenschaft auf die optionalen Werte [tcp](https://baike.baidu.com/subview/32754/8048820.htm) und [ssl](https://baike.baidu.com/view/525499.htm). Die Standardeinstellung ist tcp.

Wenn transport auf [ssl](https://baike.baidu.com/view/525499.htm) gesetzt ist, muss PHP die [openssl-Erweiterung](https://php.net/manual/zh/book.openssl.php) installiert haben.

Wenn Workerman als Client eine SSL-verschlüsselte Verbindung zum Server (HTTPS-Verbindung, WSS-Verbindung usw.) herstellt, muss diese Option wie im folgenden Beispiel auf 'ssl' gesetzt werden.

### Beispiel (HTTPS-Verbindung)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Beim Starten des Prozesses eine asynchrone Verbindung zum www.baidu.com herstellen und Daten senden, um Daten abzurufen
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Als SSL-verschlüsselte Verbindung festlegen
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Verbindung erfolgreich\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Verbindung geschlossen\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Fehlercode:$code Nachricht:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Worker ausführen
Worker::runAll();
```
