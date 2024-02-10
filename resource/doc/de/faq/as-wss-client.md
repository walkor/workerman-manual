# Als ws/wss-Client

Manchmal muss Workerman als Client mit ws/wss-Protokoll eine Verbindung zu einem Server herstellen und mit ihm interagieren. Hier ist ein Beispiel.

## Workerman als ws-Client

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Nach erfolgreichem Websocket-Handshake
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // Wenn eine Nachricht empfangen wird
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman als wss(ws+ssl)-Client

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSL erfordert den Zugriff auf Port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Setzen Sie den SSL-Verschlüsselungsmodus, um wss zu verwenden
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman als wss(ws+ssl)-Client mit lokalem SSL-Zertifikat

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Setzen Sie die lokale IP-Adresse, den Port und das SSL-Zertifikat des Zielhosts
    $context_option = array(
        // SSL-Optionen, siehe http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Lokaler Zertifikatpfad. Muss im PEM-Format sein und das lokale Zertifikat und den privaten Schlüssel enthalten.
            'local_cert'        => '/your/path/to/pemfile',
            // Passwort für die local_cert-Datei
            'passphrase'        => 'your_pem_passphrase',
            // Selbstsignierte Zertifikate zulassen
            'allow_self_signed' => true,
            // SSL-Zertifikat überprüfen
            'verify_peer'       => false
        )
    );

    // SSL erfordert den Zugriff auf Port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Setzen Sie den SSL-Verschlüsselungsmodus, um wss zu verwenden
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Weitere Einstellungen

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Bei Start des Prozesses
$worker->onWorkerStart = function()
{
    // Mit dem entfernten Websocket-Server über das Websocket-Protokoll verbinden
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Alle 55 Sekunden ein Websocket-Heartbeat mit opcode 0x9 an den Server senden
    $ws_connection->websocketPingInterval = 55;
    // Benutzerdefinierte HTTP-Header
    $ws_connection->headers = ['token' => 'value'];
    // Setzen des Datentyps. Standardmäßig ist BINARY_TYPE_BLOB für Text
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB für Text, BINARY_TYPE_ARRAYBUFFER für Binärdaten
    // Nach dem TCP-Handshake
    $ws_connection->onConnect = function($connection){
        echo "TCP-Verbindung hergestellt\n";
    };
    // Nach dem Websocket-Handshake
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Wenn eine Nachricht vom entfernten Websocket-Server empfangen wird
    $ws_connection->onMessage = function($connection, $data){
        echo "Empfangen: $data\n";
    };
    // Bei Fehlern, z.B. Verbindungsfehlern zum entfernten Websocket-Server
    $ws_connection->onError = function($connection, $code, $msg){
        echo "Fehler: $msg\n";
    };
    // Wenn die Verbindung zum entfernten Websocket-Server geschlossen wird
    $ws_connection->onClose = function($connection){
        echo "Verbindung geschlossen, versuche erneut zu verbinden\n";
        // Bei Verbindungsabbruch, nach 1 Sekunde erneut verbinden
        $connection->reConnect(1);
    };
    // Nachdem alle Rückrufe gesetzt wurden, eine Verbindung herstellen
    $ws_connection->connect();
};
Worker::runAll();
```
