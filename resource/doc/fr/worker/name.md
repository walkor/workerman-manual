# name

## Description:
```php
string Worker::$name
```

Définit le nom de l'instance actuelle du Worker, pour faciliter l'identification des processus lors de l'exécution de la commande de statut. Par défaut, s'il n'est pas défini, le nom est "none".


## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définit le nom de l'instance
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du worker...\n";
};
// Exécute le worker
Worker::runAll();
```
