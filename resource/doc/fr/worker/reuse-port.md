# reusePort
> **Remarque**
> Nécessite workerman >= 3.2.1 PHP >= 7.0, cette fonctionnalité n'est pas prise en charge par les systèmes Windows et Mac OS

## Description:

```php
bool Worker::$reusePort
```

Définit si le worker actuel active la réutilisation du port d'écoute (option SO_REUSEPORT du socket).

Lorsque la réutilisation du port d'écoute est activée, plusieurs processus sans affinité peuvent écouter le même port, et le noyau du système décide de répartir la charge pour déterminer quel processus traitera la connexion de socket, évitant ainsi l'effet de grappe, ce qui peut améliorer les performances des applications à connexions courtes multi-processus.

**Remarque :** Cette fonctionnalité nécessite une version de PHP >= 7.0

## Exemple 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Exécute le worker
Worker::runAll();
```

## Exemple 2: Écoute de plusieurs ports (protocoles multiples) avec Workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Pour chaque processus démarré, ajouter une écoute dans le processus actuel
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Plusieurs processus écoutent le même port (le socket d'écoute n'est pas hérité du processus parent)
     * La réutilisation du port doit être activée, sinon une erreur "Address already in use" sera générée
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Exécute l'écoute
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Exécute le worker
Worker::runAll();
```
