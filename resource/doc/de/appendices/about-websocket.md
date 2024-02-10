# WebSocket-Protokoll

Derzeitige Workerman WebSocket-Protokollversion ist 13.

Das WebSocket-Protokoll ist ein neues Protokoll im HTML5-Standard. Es ermöglicht eine bidirektionale Kommunikation zwischen Browser und Server.

## Beziehung zwischen WebSocket und TCP

WebSocket und HTTP sind beide Anwendungsprotokolle und basieren auf der TCP-Übertragung. WebSocket selbst hat wenig mit Sockets zu tun und kann keinesfalls gleichgesetzt werden.

## WebSocket-Handshake

Das WebSocket-Protokoll beinhaltet einen Handshake-Prozess, bei dem der Browser und der Server über das HTTP-Protokoll kommunizieren. In Workerman kann man sich so in den Handshake-Prozess einklinken.

**Für Workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Hier kann geprüft werden, ob die Verbindung legitim ist. Bei nicht legitimer Verbindung wird diese geschlossen.
        // $_SERVER['HTTP_ORIGIN'] gibt an, von welcher Website die WebSocket-Verbindung initiiert wurde.
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // In onWebSocketConnect sind $_GET und $_SERVER verfügbar
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Für Workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Übertragung von binären Daten im WebSocket-Protokoll

Das WebSocket-Protokoll kann standardmäßig nur UTF-8-Text übertragen. Für die Übertragung von binären Daten muss der folgende Abschnitt beachtet werden.

Im WebSocket-Protokoll wird ein Flag im Protokoll-Header verwendet, um zu kennzeichnen, ob binäre oder UTF-8-Textdaten übertragen werden. Der Browser überprüft, ob die Kennzeichnung und der Inhaltstyp der Übertragung übereinstimmen. Andernfalls wird die Verbindung abgebrochen.

Deshalb muss der Server beim Senden von Daten basierend auf dem Datentyp das Flag setzen. In Workerman kann für gewöhnlichen UTF-8-Text das folgende Flag gesetzt werden (normalerweise ist dieser Wert standardmäßig gesetzt und muss nicht manuell gesetzt werden):
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Für binäre Daten muss das folgende Flag gesetzt werden:
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Hinweis**: Wenn $connection->websocketType nicht gesetzt ist, ist $connection->websocketType standardmäßig BINARY_TYPE_BLOB (also der UTF-8-Texttyp). In den meisten Anwendungen wird UTF-8-Text übertragen, z. B. JSON-Daten. Daher muss $connection->websocketType normalerweise nicht manuell gesetzt werden. Nur beim Übertragen von binären Daten (z. B. Bilddaten, Protobuffer-Daten usw.) muss dieses Attribut auf BINARY_TYPE_ARRAYBUFFER gesetzt werden.

## Workerman als WebSocket-Client verwenden

Mit der [AsyncTcpConnection-Klasse](../async-tcp-connection.md) in Verbindung mit dem [WS-Protokoll](about-ws.md) kann Workerman als WebSocket-Client verwendet werden, um eine Verbindung zu einem entfernten WebSocket-Server herzustellen und eine bidirektionale Echtzeitkommunikation zu ermöglichen.
