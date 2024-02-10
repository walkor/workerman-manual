# count

## Description:
```php
int Worker::$count
```

Définit combien de processus activer pour l'instance Worker. Par défaut, c'est 1 s'il n'est pas défini.

Pour savoir comment définir le nombre de processus, veuillez consulter [**ce lien**](../faq/processes-count.md).

Remarque : cette propriété doit être définie avant l'exécution de  ```Worker::runAll();```. Cette fonctionnalité n'est pas prise en charge par le système Windows.

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Démarre 8 processus, écoute simultanément le port 8484, et fournit des services en utilisant le protocole websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du worker...\n";
};
// Exécute le worker
Worker::runAll();
```
