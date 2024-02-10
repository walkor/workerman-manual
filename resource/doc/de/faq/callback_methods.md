# PHP verschiedene Rückrufmethoden

Das Schreiben von Rückrufen als anonyme Funktionen ist die bequemste Methode in PHP. Abgesehen von der Verwendung anonymer Funktionen gibt es jedoch noch andere Methoden, um Rückrufe in PHP zu schreiben. Im Folgenden sind Beispiele für verschiedene Rückrufmethoden in PHP aufgeführt.

## 1. Rückruf als anonymer Funktion

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Anonymer Funktionsrückruf
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Senden von "Hallo Welt" an den Browser
    $connection->send('Hallo Welt');
};

Worker::runAll();
```

## 2. Rückruf als reguläre Funktion

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Rückruf als reguläre Funktion
$http_worker->onMessage = 'on_message';

// Reguläre Funktion
function on_message(TcpConnection $connection, Request $request)
{
    // Senden von "Hallo Welt" an den Browser
    $connection->send('Hallo Welt');
}

Worker::runAll();
```

## 3. Verwendung von Klassenmethoden als Rückruf

MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```

Startskript start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass laden
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Objekt erstellen
$my_object = new MyClass();

// Klassenmethoden aufrufen
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Hinweis: Die oben gezeigte Codestruktur erlaubt keine Initialisierung von Ressourcen (MySQL-Verbindung, Redis-Verbindung, Memcache-Verbindung usw.) im Konstruktor, da ```$my_object = new MyClass();``` im Hauptprozess ausgeführt wird. Zum Beispiel wird im Hauptprozess initialisierte MySQL-Verbindung und ähnliche Ressourcen von den Kindprozessen geerbt, und jeder Kindprozess kann auf diese Datenbankverbindung zugreifen. Diese Verbindungen befinden sich jedoch auf demselben Server auf der MySQL-Seite, was zu unvorhersehbaren Fehlern führen kann, wie z.B. dem Fehler "mysql gone away".

Wenn die oben gezeigte Codestruktur Ressourcen im Konstruktor der Klasse initialisieren muss, kann die folgende Methode verwendet werden.

MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Angenommen, die Datenbankverbindungsklasse ist MyDbClass
        $this->db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```

Startskript start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Klasseninitialisierung in onWorkerStart
$worker->onWorkerStart = function($worker) {
    // MyClass laden
    require_once __DIR__.'/MyClass.php';
    
    // Objekt erstellen
    $my_object = new MyClass();

    // Klassenmethoden aufrufen
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

In der obigen Codestruktur wird onWorkerStart im Kindprozess ausgeführt, was bedeutet, dass jeder Kindprozess seine eigene MySQL-Verbindung herstellt, ohne dass es zu gemeinsamen Verbindungen kommt. Außerdem unterstützt diese Methode das Neuladen des Geschäftslogikcodes. Da MyClass.php im Kindprozess geladen wird, kann eine Änderung am Geschäftslogikcode in MyClass.php direkt mit einem Reload wirksam werden.

## 4. Verwendung von statischen Klassenmethoden als Rückruf

Statistische Klasse MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```

Startskript start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass laden
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Aufrufen von statischen Klassenmethoden
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Wenn die Klasse einen Namensraum hat, erfolgt der Aufruf folgendermaßen
// $worker->onWorkerStart = array('dein\namensraum\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('dein\namensraum\MyClass', 'onConnect');
// $worker->onMessage     = array('dein\namensraum\MyClass', 'onMessage');
// $worker->onClose       = array('dein\namensraum\MyClass', 'onClose');
// $worker->onWorkerStop  = array('dein\namensraum\MyClass', 'onWorkerStop');

Worker::runAll();
```

Hinweis: Gemäß dem Funktionsprinzip von PHP werden Konstruktoren nicht aufgerufen, es sei denn, es wird "new" verwendet, und statische Klassenmethoden dürfen keine Verwendung von ```$this``` enthalten.
