# onWorkerStart
## Description:
```php
callback Worker::$onWorkerStart
```

Définit la fonction de rappel lors du démarrage du processus de travail, qui sera exécutée à chaque démarrage du processus de travail.

Remarque : onWorkerStart est exécuté lorsque le processus enfant démarre. Si plusieurs processus enfants sont démarrés (```$worker->count > 1```), chaque processus enfant sera exécuté une fois, soit un total de ```$worker->count``` fois.


## Paramètres de la fonction de rappel

 ``` $worker ```

Il s'agit de l'objet Worker.


## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Démarrage du processus de travail...\n";
};
// Lance le processus de travail
Worker::runAll();
```

Remarque : En plus d'utiliser une fonction anonyme comme rappel, il est également possible de [consulter ici](../faq/callback_methods.md) d'autres méthodes de rappel.
