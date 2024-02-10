# Konstruktor __construct

## Erklärung:
```php
Worker::__construct([string $listen , array $context])
```

Initialisiert eine Instanz des Worker-Containers und ermöglicht die Festlegung einiger Eigenschaften und Rückruf-Schnittstellen, um spezifische Funktionen zu erfüllen.

## Parameter
#### **``` $listen ```** (optional, wenn nicht angegeben, bedeutet es, dass kein Port überwacht wird)


Wenn der Parameter ```$listen``` festgelegt ist, wird der Socket überwacht.

Das Format von $listen ist <Protokoll>://<Adresse>

**<Protokoll> Kann in folgenden Formaten angegeben werden:**


TCP: z.B. ```tcp://0.0.0.0:8686```

UDP: z.B. ```udp://0.0.0.0:8686```

Unix: z.B. ```unix:///tmp/my_file``` ```(benötigt Workerman>=3.2.7)```

HTTP: z.B. ```http://0.0.0.0:80```

WebSocket: z.B. ```websocket://0.0.0.0:8686```

Text: z.B. ```text://0.0.0.0:8686``` ```(Text ist ein in Workerman integriertes Textprotokoll, das mit Telnet kompatibel ist. Weitere Einzelheiten finden Sie im Anhang des Textprotokolls)```

und andere benutzerdefinierte Protokolle, siehe Anhang zur Anpassung von Kommunikationsprotokollen in diesem Handbuch


**<Adresse> Kann in folgenden Formaten angegeben werden:**


Wenn es sich um einen Unix-Socket handelt, ist die Adresse ein lokaler Dateipfad

Für Nicht-Unix-Sockets ist das Adressformat <Lokale IP>:<Port>

<Lokale IP> kann ```0.0.0.0``` sein, um auf allen Netzwerkschnittstellen des lokalen Rechners zu hören, einschließlich der internen und externen IP-Adressen sowie der lokalen Schleife 127.0.0.1

<Lokale IP> Wenn ```127.0.0.1``` ist, dann hört es nur auf die lokale Schleife und ist nur vom lokalen Rechner aus erreichbar, nicht von externen Quellen

<Lokale IP> Wenn es sich um eine interne IP-Adresse handelt, ähnlich wie ```192.168.xx.xx```, dann hört es nur auf die interne IP-Adresse, sodass externe Benutzer keine Verbindung herstellen können

Wenn der Wert von <Lokale IP> nicht zu einer lokalen IP-Adresse gehört, kann keine Überwachung durchgeführt werden, und es wird ein Fehler ```Cannot assign requested address``` angezeigt

**Anmerkung:** Der <Port> darf nicht größer als 65535 sein. Wenn <Port> kleiner als 1024 ist, ist root-Berechtigung erforderlich, um zuzuhören. Der überwachte Port muss ein inländischer ungenutzter Port des lokalen Computers sein, ansonsten kann nicht zugehört werden und es wird ein Fehler ```Address already in use``` angezeigt.


#### **``` $context ```**


Ein Array. Wird verwendet, um die Kontextoptionen des Sockets zu übergeben. Siehe [Socket-Kontextoptionen](https://php.net/manual/zh/context.socket.php)


## Beispiel

Worker dient als HTTP-Container, um HTTP-Anfragen zu verarbeiten
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```


Worker dient als WebSocket-Container, um WebSocket-Anfragen zu verarbeiten
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```


Worker dient als TCP-Container, um TCP-Anfragen zu verarbeiten
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```


Worker dient als UDP-Container, um UDP-Anfragen zu verarbeiten
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```


Worker überwacht Unix-Domain-Socket ```(Workerman-Version >=3.2.7 erforderlich)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Worker ausführen
Worker::runAll();
```


Ein Worker-Container, der keine Überwachung ausführt, wird verwendet, um zeitgesteuerte Aufgaben zu verarbeiten
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Führt alle 2,5 Sekunden aus
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Worker ausführen
Worker::runAll();
```


**Worker lauscht auf einem Port mit benutzerdefiniertem Protokoll**

Endgültige Verzeichnisstruktur
```
├── Protocols              // Dies ist das zu erstellende Protokollverzeichnis
│   └── MyTextProtocol.php // Dies ist die zu erstellende benutzerdefinierte Protokolldatei
├── test.php  // Dies ist das zu erstellende Testskript
└── Workerman // Workerman-Quellcodeverzeichnis, in dem der Code nicht verändert werden soll
```

1. Erstellen Sie das Verzeichnis Protocols und erstellen Sie eine Protokolldatei
Protocols/MyTextProtocol.php (gemäß der oben genannten Verzeichnisstruktur)

```php
// Der Namespace des benutzerdefinierten Protokolls ist immer Protocols
namespace Protocols;
// Einfaches Textprotokoll, Protokollformat ist Text+Zeilenwechsel
class MyTextProtocol
{
    // Funktion für die Paketaufteilung, gibt die Länge des aktuellen Pakets zurück
    public static function input($recv_buffer)
    {
        // Sucht nach dem Zeilenwechsel
        $pos = strpos($recv_buffer, "\n");
        // Wenn kein Zeilenwechsel gefunden wird, handelt es sich um kein vollständiges Paket, daher wird 0 zurückgegeben und auf weitere Daten gewartet
        if($pos === false)
        {
            return 0;
        }
        // Ist ein Zeilenwechsel vorhanden, wird die Länge des aktuellen Pakets zurückgegeben, einschließlich des Zeilenwechsels
        return $pos+1;
    }

    // Nach dem Empfangen eines vollständigen Pakets wird automatisch MyTextProtocol::decode('Empfangene Daten') ausgeführt und das Ergebnis über $data an die onMessage-Funktion übergeben
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Bevor Daten an den Client gesendet werden, erfolgt automatisch eine Kodierung durch encode und dann wird es an den Client gesendet. Hier wird ein Zeilenwechsel hinzugefügt
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Verwenden Sie das Protokoll MyTextProtocol, um Anfragen zu überwachen und zu verarbeiten

Entsprechend der oben genannten endgültigen Verzeichnisstruktur erstellen Sie die Datei test.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol-Worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Nach dem Empfangen vollständiger Daten (Ende mit Zeilenwechsel) wird automatisch MyTextProtocol::decode('Erhaltene Daten') ausgeführt
 * Das Ergebnis wird über $data an das onMessage-Callback übergeben
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Daten an den Client senden. Automatisch wird MyTextProtocol::encode('hello world') zur Protokollkodierung aufgerufen
     * und dann an den Client gesendet
     */
    $connection->send("hello world");
};

// Alle Worker ausführen
Worker::runAll();
```

3. Testen

Öffnen Sie das Terminal, wechseln Sie in das Verzeichnis, in dem sich die Datei test.php befindet, und führen Sie ```php test.php start``` aus
```sh
php test.php start
Workerman[test.php] start im DEBUG-Modus
----------------------- WORKERMAN -----------------------------
Workerman-Version: 3.2.7          PHP-Version: 5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Drücken Sie Strg+C, um zu beenden. Erfolgreich gestartet.
```

Öffnen Sie das Terminal und testen Sie mit Telnet (es wird empfohlen, Telnet unter einem Linux-System zu verwenden)

Angenommen, es handelt sich um einen lokalen Test
Führen Sie im Terminal ``` telnet 127.0.0.1 5678 ``` aus
Dann geben Sie hi ein und drücken Sie Enter
Sie erhalten "hello world" als Antwort
```sh
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
