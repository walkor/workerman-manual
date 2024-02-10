# Explication
À partir de la version 4.x, Workerman a renforcé son support des services HTTP. Il a introduit une classe de requête, une classe de réponse, une classe de session et le [SSE](SSE.md). Si vous souhaitez utiliser les services HTTP de Workerman, il est fortement recommandé d'utiliser la version 4.x ou une version ultérieure.

**Veuillez noter que les exemples ci-dessous sont tous basés sur l'utilisation de la version 4.x de Workerman et ne sont pas compatibles avec la version 3.x.**

## Changer le moteur de stockage de session
Workerman offre un moteur de stockage de session basé sur les fichiers et un moteur de stockage de session basé sur Redis. Le moteur par défaut est basé sur les fichiers. Si vous souhaitez passer à un moteur de stockage basé sur Redis, veuillez suivre le code ci-dessous.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Configuration Redis
$config = [
    'host'     => '127.0.0.1', // Paramètre obligatoire
    'port'     => 6379,        // Paramètre obligatoire
    'timeout'  => 2,           // Paramètre facultatif
    'auth'     => '******',    // Paramètre facultatif
    'database' => 1,           // Paramètre facultatif
    'prefix'   => 'session_'   // Paramètre facultatif
];
// Utiliser la méthode Session::handlerClass pour changer la classe de pilote de session sous-jacente
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Définir l'emplacement de stockage de session
Lorsque le moteur de stockage par défaut est utilisé, les données de session sont stockées par défaut sur le disque à l'emplacement renvoyé par `session_save_path()`. Vous pouvez utiliser la méthode suivante pour modifier l'emplacement de stockage.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Définir l'emplacement du fichier de session
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'Tom');
    $connection->send($session->get('name'));
};

// Exécuter le worker
Worker::runAll();
```
## Nettoyage des fichiers de session
Lorsque le moteur de stockage de session par défaut est utilisé, plusieurs fichiers de session peuvent être présents sur le disque. Workerman nettoie les fichiers de session expirés en fonction des options `session.gc_probability`, `session.gc_divisor` et `session.gc_maxlifetime` définies dans php.ini. Pour plus d'informations sur ces trois options, consultez le [manuel de PHP](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)

## Changer de pilote de stockage
En plus du moteur de stockage de session basé sur les fichiers et le moteur de stockage de session basé sur Redis, Workerman vous permet d'ajouter de nouveaux moteurs de stockage de session en implémentant l'interface standard [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php), tels que le moteur de stockage de session MangoDb ou le moteur de stockage de session MySQL, etc.

**Processus d'ajout d'un nouveau moteur de stockage de session**
 1. Implementer l'interface [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)
 2. Utiliser la méthode `Workerman\Protocols\Http\Session::handlerClass($nom_de_classe, $config)` pour remplacer l'interface sous-jacente SessionHandler
 
** Implémentation de l'interface SessionHandlerInterface **

Le pilote de stockage de session personnalisé doit implémenter l'interface SessionHandlerInterface. Cette interface comprend les méthodes suivantes:
```php
SessionHandlerInterface {
    /* Méthodes */
    abstract public lire ( string $session_id ) : string
    abstract public ecrire ( string $session_id, string $session_data ) : bool
    abstract public detruire ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public ouvrir ( string $save_path, string $session_name ) : bool
}
```
**Explication de SessionHandlerInterface **
 - La méthode lire est utilisée pour lire toutes les données de session correspondant à l'identifiant de session depuis le stockage. Ne pas désérialiser les données, car le framework le fera automatiquement.
 - La méthode ecrire est utilisée pour écrire les données de session correspondant à l'identifiant de session donné dans le stockage. Ne pas sérialiser les données, car le framework le fait déjà automatiquement.
 - La méthode detruire est utilisée pour détruire les données de session correspondant à l'identifiant de session donné.
 - La méthode gc est utilisée pour supprimer les données de session expirées. Le stockage devrait supprimer toutes les sessions dont le dernier accès est supérieur à maxlifetime.
 - close ne nécessite aucune action, il suffit de retourner true.
 - open ne nécessite aucune action, il suffit de retourner true.

**Remplacement du pilote sous-jacent**

Une fois l'interface SessionHandlerInterface implémentée, utilisez la méthode suivante pour changer de pilote de stockage de session :
```php
Workerman\Protocols\Http\Session::handlerClass($nom_de_classe, $config);
```
 - $nom_de_classe est le nom de la classe implémentant l'interface SessionHandlerInterface. Si le namespace est utilisé, le namespace complet doit être inclus.
 - $config est les paramètres du constructeur de la classe SessionHandler

**Implémentation spécifique**

*Remarque : la classe MySessionHandler est uniquement utilisée à titre d'exemple pour illustrer le processus de changement du pilote de stockage de session. MySessionHandler ne doit pas être utilisé en environnement de production.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function lire($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function écrire($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function detruire($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

//  Supposons que la nouvelle classe MySessionHandler a besoin de certaines configurations
$config = ['host' => 'localhost'];
//  Utiliser Workerman\Protocols\Http\Session::handlerClass($nom_de_classe, $config) pour changer la classe de pilote de session sous-jacente
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
