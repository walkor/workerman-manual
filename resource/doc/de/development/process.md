# Grundlegender Ablauf
(Am Beispiel eines einfachen WebSocket-Chatroom-Servers)

#### 1. Projektverzeichnis an beliebiger Stelle erstellen
z.B. SimpleChat/
Geben Sie im Verzeichnis den Befehl `composer require workerman/workerman` ein.

#### 2. Einbindung von `vendor/autoload.php` (nach der Installation von Composer erstellt)
Erstellen Sie `start.php` und binden Sie `vendor/autoload.php` ein.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Auswahl des Protokolls
Hier wählen wir das Text-Protokoll aus (ein von Workerman definiertes Protokoll im Format Text + Zeilenumbruch).

(Aktuell unterstützt Workerman HTTP, WebSocket und das Text-Protokoll. Wenn ein anderes Protokoll benötigt wird, befolgen Sie das Kapitel zum Entwickeln Ihres eigenen Protokolls.)

#### 4. Erstellen Sie das Einstiegsstartskript entsprechend Ihren Anforderungen
Das folgende Beispiel ist der Einstieg in eine einfache Chatroom-Anwendung.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Wenn ein Client verbunden ist, wird die UID zugewiesen, die Verbindung wird gespeichert und alle Clients werden benachrichtigt.
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Weist dieser Verbindung eine UID zu.
    $connection->uid = ++$global_uid;
}

// Wenn ein Client eine Nachricht sendet, wird diese an alle weitergeleitet.
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("Benutzer[{$connection->uid}] sagte: $data");
    }
}

// Wenn ein Client die Verbindung trennt, wird dies allen Clients mitgeteilt.
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("Benutzer[{$connection->uid}] hat sich abgemeldet");
    }
}

// Erstellt einen Worker für das Textprotokoll, der den Port 2347 überwacht.
$text_worker = new Worker("text://0.0.0.0:2347");

// Es wird nur ein Prozess gestartet, um die Datenübertragung zwischen den Clients zu erleichtern.
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Testen
Text-Protokoll kann mit dem Telnet-Befehl getestet werden.
```shell
telnet 127.0.0.1 2347
```
