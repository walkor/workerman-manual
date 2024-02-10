# __construct Methode
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Erstellt ein asynchrones Verbindungobjekt.

AsyncTcpConnection ermöglicht es Workerman, als Client eine asynchrone Verbindung zu einem Remote-Server herzustellen und Daten auf der Verbindung asynchron über die send-Schnittstelle und das onMessage-Callback zu senden und zu verarbeiten.

## Parameter
Parameter: ``` remote_address ```

Die Adresse der Verbindung, zum Beispiel
``` tcp://www.baidu.com:80 ```
``` ssl://www.baidu.com:443 ```
``` ws://echo.websocket.org:80 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

Parameter: ``` $context_option ```

``` Dieser Parameter erfordert (workerman >= 3.3.5) ```

Es wird verwendet, um den Socket-Kontext einzurichten, beispielsweise durch die Verwendung von ```bindto```, um festzulegen, von welcher (Netzwerkkarte) IP und welchem Port aus auf das externe Netzwerk zugegriffen werden soll, oder um SSL-Zertifikate festzulegen.

Siehe [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Socket-Kontextoptionen](https://php.net/manual/zh/context.socket.php), [SSL-Kontextoptionen](https://php.net/manual/zh/context.ssl.php).

## Hinweis

Derzeit unterstützt AsyncTcpConnection die Protokolle [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

Es unterstützt auch benutzerdefinierte Protokolle, siehe [Wie man benutzerdefinierte Protokolle definiert](../protocols/how-protocols.md).

SSL erfordert Workerman >= 3.3.4 und die Installation der [openssl extension](https://php.net/manual/zh/book.openssl.php).

AsyncTcpConnection unterstützt derzeit nicht das [http](https://baike.baidu.com/view/9472.htm)-Protokoll.

Sie können ```new AsyncTcpConnection('ws://...')``` verwenden, um wie im Browser eine Verbindung zu einem entfernten WebSocket-Server über WebSocket herzustellen. Siehe [Beispiel](../appendices/about-ws.md). Es ist jedoch nicht möglich, eine WebSocket-Verbindung in Workerman mit ```new AsyncTcpConnection('websocket://...')``` herzustellen.

## Beispiele

### Beispiel 1: Asynchroner Zugriff auf einen externen HTTP-Service
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Bei Prozessstart wird eine Verbindung zu www.baidu.com hergestellt und Daten gesendet, um Daten abzurufen
$task->onWorkerStart = function($task)
{
    // Direkte Angabe von http wird nicht unterstützt, aber HTTP-Protokolldaten können über TCP gesendet werden
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Wenn die Verbindung erfolgreich hergestellt wurde, senden Sie die HTTP-Anforderungsdaten
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

### Beispiel 2: Asynchroner Zugriff auf einen externen WebSocket-Service und Festlegen des lokalen IP- und Portzugriffs
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Festlegen der lokalen IP und Port des Hosts, der zugreifen soll (jede Socket-Verbindung belegt einen lokalen Port)
    $context_option = array(
        'socket' => array(
            // Die IP muss die IP der lokalen Netzwerkkarte sein und zum Host zugreifen können, sonst wird sie ungültig
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('Hallo');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Beispiel 3: Asynchroner Zugriff auf einen externen wss-Port und Festlegen eines lokalen SSL-Zertifikats
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Festlegen der lokalen IP und des Ports, von denen aus auf den Host zugegriffen werden sowie des SSL-Zertifikats
    $context_option = array(
        'socket' => array(
            // Die IP muss die IP der lokalen Netzwerkkarte sein und zum Host zugreifen können, sonst wird sie ungültig
            'bindto' => '114.215.84.87:2333',
        ),
        // SSL-Optionen, siehe https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Pfad zum lokalen Zertifikat. Muss im PEM-Format sein und das lokale Zertifikat sowie den privaten Schlüssel enthalten.
            'local_cert'        => '/your/path/to/pemfile',
            // Passwort für die local_cert-Datei.
            'passphrase'        => 'your_pem_passphrase',
            // Erlaubt selbstsignierte Zertifikate.
            'allow_self_signed' => true,
            // Überprüfung des SSL-Zertifikats erforderlich.
            'verify_peer'       => false
        )
    );

    // Asynchrone Verbindung aufbauen
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // SSL-Verschlüsselung für den Zugriff festlegen
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('Hallo');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
