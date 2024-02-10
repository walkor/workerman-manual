# protocole
Exige ```(workerman >= 3.2.7)```

## Description:
```php
string Worker::$protocol
```

Définit la classe de protocole pour l'instance actuelle du Worker.

Remarque : la classe de traitement du protocole peut être spécifiée directement lors de l'initialisation du Worker dans les paramètres d'écoute. Par exemple,
```php
$worker = new Worker('http://0.0.0.0:8686');
```

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Exécute le worker
Worker::runAll();
```

Le code ci-dessus est équivalent au code ci-dessous

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Vérifie tout d'abord si l'utilisateur a une classe de protocole personnalisée \Protocols\Http,
 * sinon utilise la classe de protocole intégrée de workerman Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Exécute le worker
Worker::runAll();
```
