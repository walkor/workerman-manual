# onWorkerReload
Required `(workerman >= 3.2.5)`
## Description:
```php
callback Worker::$onWorkerReload
```
Cette fonctionnalité est rarement utilisée.

Définit le rappel à exécuter par le Worker lorsqu'il reçoit un signal de rechargement.

Le rappel onWorkerReload peut être utilisé pour effectuer de nombreuses tâches, telles que recharger un fichier de configuration métier sans redémarrer le processus.

**Remarque**:

Par défaut, l'action du processus fils après avoir reçu le signal de rechargement est de se terminer et de redémarrer, afin que le nouveau processus recharge le code métier pour effectuer la mise à jour du code. Ainsi, il est normal que le processus fils quitte immédiatement après avoir exécuté le rappel onWorkerReload.

Si vous souhaitez que le processus fils n'exécute que le rappel onWorkerReload après avoir reçu le signal de rechargement, sans quitter, vous pouvez définir la propriété reloadable de l'instance correspondante de Worker sur false lors de l'initialisation de l'instance Worker.


## Paramètres de la fonction de rappel

 ``` $worker ```

Il s'agit de l'objet Worker.


## Exemple


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définir reloadable sur false, ce qui signifie que le processus fils n'exécutera pas de redémarrage en cas de signal de rechargement
$worker->reloadable = false;
// Après le rechargement, informer tous les clients que le serveur a été rechargé
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Exécuter le worker
Worker::runAll();
```

Remarque: En plus d'utiliser une fonction anonyme comme rappel, d'autres façons de définir le rappel peuvent être trouvées [ici](../faq/callback_methods.md).
