# pidFile
## Description:
```php
static string Worker::$pidFile
```

Sauf indication contraire, il est recommandé de ne pas définir cette propriété.

Cette propriété est une propriété statique globale utilisée pour définir le chemin du fichier pid du processus WorkerMan.

Cette configuration est utile pour la surveillance, par exemple en plaçant le fichier pid de WorkerMan dans un répertoire spécifique, cela facilite la lecture par certains logiciels de surveillance pour surveiller l'état du processus WorkerMan. 

Si cette propriété n'est pas définie, WorkerMan générera automatiquement un fichier pid à un emplacement par défaut (notez que avant la version 3.2.3, cela se faisait par défaut dans ```sys_get_temp_dir()```) et pour éviter les conflits de pid dus au démarrage de plusieurs instances de WorkerMan, le fichier pid généré contient le chemin actuel de WorkerMan.

Remarque : Cette propriété doit être définie avant l'exécution de ```Worker::runAll();```. Cette fonctionnalité n'est pas supportée sur les systèmes Windows.

## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du worker";
};
// Exécution du worker
Worker::runAll();
```
