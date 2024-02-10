# Einfaches Entwicklungsbeispiel

## Installation

**Installieren von Workerman**
Führen Sie in einem leeren Verzeichnis den Befehl aus:
`composer require workerman/workerman`

## Beispiel 1: Bereitstellung von Webdiensten nach außen mit dem HTTP-Protokoll
**Erstellen Sie die Datei start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Erstellen eines Workers, der den Port 2345 überwacht und das HTTP-Protokoll verwendet
$http_worker = new Worker("http://0.0.0.0:2345");

// Starten von 4 Prozessen zur Bereitstellung von Diensten nach außen
$http_worker->count = 4;

// Antwort an den Browser mit "hello world", wenn Daten vom Browser empfangen werden
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // "hello world" an den Browser senden
    $connection->send('hello world');
};

// Worker ausführen
Worker::runAll();
```

**Befehlszeile ausführen (für Windows-Anwender über die [cmd-Befehlszeile](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn) usw.)**
```shell
php start.php start
```

**Testen**

Angenommen, die Server-IP lautet 127.0.0.1

Besuchen Sie die URL http://127.0.0.1:2345 im Browser.

**Hinweis:**

1. Wenn Sie auf Probleme beim Zugriff stoßen, überprüfen Sie die [Gründe für ein erfolgloses Client-Verbinden](../faq/client-connect-fail.md).

2. Der Server verwendet das HTTP-Protokoll und kann nur mit dem HTTP-Protokoll kommunizieren. Die Kommunikation mit anderen Protokollen wie WebSocket ist nicht direkt möglich.

## Beispiel 2: Bereitstellung von Diensten nach außen mit dem WebSocket-Protokoll
**Erstellen Sie die Datei ws_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Beachten Sie: Hier wird im Gegensatz zum vorherigen Beispiel das WebSocket-Protokoll verwendet
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Starten von 4 Prozessen zur Bereitstellung von Diensten nach außen
$ws_worker->count = 4;

// Beim Empfangen von Daten vom Client, "hello $data" an den Client zurücksenden
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // "hello $data" an den Client senden
    $connection->send('hello ' . $data);
};

// Worker ausführen
Worker::runAll();
```

**Befehlszeile ausführen**
```shell
php ws_test.php start
```

**Testen**

Öffnen Sie den Chrome-Browser und drücken Sie F12, um die Entwicklertools zu öffnen. Geben Sie im Bereich "Console" Folgendes ein (oder fügen Sie den unten stehenden Code in eine HTML-Seite ein und führen Sie ihn mit JavaScript aus)

```javascript
// Angenommen, die Server-IP lautet 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Verbindung erfolgreich");
    ws.send('tom');
    alert("Sende einen String 'tom' an den Server");
};
ws.onmessage = function(e) {
    alert("Nachricht vom Server erhalten: " + e.data);
};
```

**Hinweis:**

1. Wenn Sie auf Probleme beim Zugriff stoßen, überprüfen Sie die [Gründe für ein erfolgloses Client-Verbinden](../faq/client-connect-fail.md).

2. Der Server verwendet das WebSocket-Protokoll und kann nur mit dem WebSocket-Protokoll kommunizieren. Die Kommunikation mit anderen Protokollen wie HTTP ist nicht direkt möglich.

## Beispiel 3: Direkte Verwendung der TCP-Datenübertragung
**Erstellen Sie die Datei tcp_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Erstellen eines Workers, der den Port 2347 überwacht und kein Anwendungsprotokoll verwendet
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Starten von 4 Prozessen zur Bereitstellung von Diensten nach außen
$tcp_worker->count = 4;

// Beim Empfangen von Daten vom Client
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // "hello $data" an den Client senden
    $connection->send('hello ' . $data);
};

// Worker ausführen
Worker::runAll();
```

**Befehlszeile ausführen**
```shell
php tcp_test.php start
```

**Testen: Befehlszeile ausführen**
(Dies ist die Auswirkung der Befehlszeile in Linux. In Windows sieht es etwas anders aus)

```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Verbunden mit 127.0.0.1.
Escape-Zeichenfolge ist '^]'.
tom
hello tom
```

**Hinweis:**

1. Wenn Sie auf Probleme beim Zugriff stoßen, überprüfen Sie die [Gründe für ein erfolgloses Client-Verbinden](../faq/client-connect-fail.md).

2. Der Server verwendet das nackte TCP-Protokoll, daher ist eine direkte Kommunikation mit Protokollen wie WebSocket und HTTP nicht möglich.
