# reloadable
## Description:
```php
bool Worker::$reloadable
```

Lors de l'exécution de `php start.php reload`, un signal de rechargement (SIGUSR1) sera envoyé à tous les processus enfants.

Une fois qu'un processus enfant reçoit le signal de rechargement, il se terminera automatiquement et le processus principal démarrera automatiquement un nouveau processus, généralement utilisé pour mettre à jour le code métier.

Lorsque la propriété $reloadable du processus est définie sur false, la réception du signal de rechargement ne déclenchera que [onWorkerReload](on-worker-reload.md), sans redémarrer le processus actuel.

Par exemple, dans le modèle Gateway/Worker, le processus gateway est responsable de la gestion des connexions client, tandis que le processus worker est responsable du traitement des demandes. En définissant la propriété reloadable du processus gateway sur false, il est possible de mettre à jour le code métier sans interrompre les connexions client lors du rechargement.

## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définir si cette instance redémarre après avoir reçu le signal de rechargement
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du travailleur...\n";
};
// Exécuter le travailleur
Worker::runAll();
```
