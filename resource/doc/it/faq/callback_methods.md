# Alcune modalità di callback in PHP
In PHP, l'utilizzo di funzioni anonime è il modo più conveniente per la scrittura di callback, ma oltre al metodo con funzioni anonime, PHP offre anche altri modi per gestire le callback. Di seguito sono riportati alcuni esempi di modalità di callback in PHP.

## 1. Callback con funzione anonima
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback con funzione anonima
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Invia 'hello world' al browser
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Callback con funzione normale
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback con funzione normale
$http_worker->onMessage = 'on_message';

// Funzione normale
function on_message(TcpConnection $connection, Request $request)
{
    // Invia 'hello world' al browser
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Metodo di classe come callback
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
Script di avvio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Carica MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Crea un oggetto
$my_object = new MyClass();

// Chiama il metodo della classe
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Nota: La struttura del codice sopra non consente di inizializzare risorse (connessioni MySQL, connessioni Redis, connessioni Memcache, ecc.) nel costruttore, poiché ```$my_object = new MyClass();``` viene eseguito nel processo principale. Ad esempio, con MySQL, se si inizializzano le connessioni nel processo principale, tali risorse verranno ereditate dai processi figlio e ognuno potrà operare su quella connessione al database. Tuttavia, poiché tali connessioni corrispondono a una singola connessione sul lato server di MySQL, si verificheranno errori imprevisti, come ad esempio l'errore ```mysql gone away```.

Se è necessario inizializzare le risorse nel costruttore della classe, è possibile utilizzare la seguente sintassi.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Supponiamo che la classe di connessione al database sia MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){ /* ... */ }
    public function onConnect(TcpConnection $connection){ /* ... */ }
    public function onMessage(TcpConnection $connection, $message) { /* ... */ }
    public function onClose(TcpConnection $connection){ /* ... */ }
    public function onWorkerStop(Worker $worker){ /* ... */ }
}
```
Script di avvio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Inizializza la classe in onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Carica MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Crea un oggetto
    $my_object = new MyClass();

    // Chiama i metodi della classe
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

Nella struttura del codice sopra, onWorkerStart viene eseguito nel processo figlio, quindi ciascun processo figlio stabilisce la propria connessione MySQL e non vi è condivisione di connessioni. Questo approccio supporta anche il reload del codice di business. Poiché MyClass.php viene caricato nel processo figlio, è sufficiente eseguire un reload per rendere effettive le modifiche del codice di business a MyClass.php.

## 4. Utilizzo di un metodo statico della classe come callback
Classe statica MyClass.php
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
Script di avvio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Carica MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Chiama i metodi statici della classe.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Se la classe ha un namespace, il metodo è chiamato in questo modo
// $worker->onWorkerStart = array('tuo\namespace\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('tuo\namespace\MyClass', 'onConnect');
// $worker->onMessage     = array('tuo\namespace\MyClass', 'onMessage');
// $worker->onClose       = array('tuo\namespace\MyClass', 'onClose');
// $worker->onWorkerStop  = array('tuo\namespace\MyClass', 'onWorkerStop');

Worker::runAll();
```

Nota: In base al meccanismo di esecuzione di PHP, se non viene utilizzata la parola chiave "new", il costruttore non verrà chiamato e inoltre nei metodi di classe statica non è consentito utilizzare ```$this```.
