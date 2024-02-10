# Quelques façons d'écrire des rappels en PHP

En PHP, l'utilisation de fonctions anonymes pour écrire des rappels est la plus pratique, mais en plus de l'utilisation de fonctions anonymes, PHP propose d'autres façons d'écrire des rappels. Voici des exemples de quelques façons d'écrire des rappels en PHP.

## 1. Rappel avec une fonction anonyme
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Rappel avec une fonction anonyme
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Envoyer "hello world" au navigateur
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Rappel avec une fonction ordinaire
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Rappel avec une fonction ordinaire
$http_worker->onMessage = 'on_message';

// Fonction ordinaire
function on_message(TcpConnection $connection, Request $request)
{
    // Envoyer "hello world" au navigateur
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Utilisation d'une méthode de classe comme rappel
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
Script de démarrage start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Inclusion de MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Création d'un objet
$my_object = new MyClass();

// Appel des méthodes de la classe
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

**Remarque：**
La structure de code ci-dessus ne permet pas d'initialiser des ressources (connexion MySQL, connexion Redis, connexion Memcache, etc.) dans le constructeur, car ```$my_object = new MyClass();``` s'exécute dans le processus principal. Prenons l'exemple de MySQL, les connexions MySQL initialisées dans le processus principal seront héritées par les sous-processus et chaque sous-processus pourra utiliser cette connexion à la base de données. Cependant, ces connexions correspondent à une seule connexion du côté serveur MySQL, ce qui entraînera des erreurs imprévisibles, comme l'erreur "mysql gone away".

Si la structure du code ci-dessus nécessite d'initialiser des ressources dans le constructeur de la classe, vous pouvez adopter l'approche suivante.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Supposons que la classe de connexion à la base de données est MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Script de démarrage start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Initialiser dans onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Inclusion de MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Création d'un objet
    $my_object = new MyClass();

    // Appel des méthodes de la classe
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

Dans la structure de code ci-dessus, onWorkerStart s'exécute déjà dans le sous-processus, ce qui signifie que chaque sous-processus établit sa propre connexion MySQL, éliminant ainsi le problème de connexion partagée. De plus, cela permet de prendre en charge le rechargement du code métier. Comme MyClass.php est chargé dans le sous-processus, toute modification apportée à MyClass.php dans les règles de rechargement sera directement prise en compte.

## 4. Utilisation d'une méthode statique de classe comme rappel
MyClass.php statique
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
Script de démarrage start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Inclusion de MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Appel des méthodes statiques de la classe.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Si la classe a un espace de noms, cela serait écrit de la manière suivante :
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

**Remarque：** Selon le fonctionnement de PHP, si `new` n'est pas utilisé, le constructeur ne sera pas appelé. De plus, les méthodes de classe statiques ne peuvent pas utiliser ```$this```.
