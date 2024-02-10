# Connexions
## Description:
```php
array Worker::$connections
```

Au format
```php
array(id=>connexion, id=>connexion, ...)
```

Cette propriété stocke tous les objets de connexion client du **processus actuel**, avec l'identifiant comme clé et l'objet de connexion comme valeur. Consultez le manuel pour plus de détails sur la propriété [id de TcpConnection](../tcp-connection/id.md).

```$connections``` est très utile lors de la diffusion ou de l'obtention d'un objet de connexion en fonction de son ID.

Si l'ID de la connexion est connu sous la forme ```$id```, il est possible d'obtenir l'objet de connexion correspondant de manière pratique en utilisant ```$worker->connections[$id]```, permettant ainsi de manipuler la connexion socket correspondante, par exemple en envoyant des données via ```$worker->connections[$id]->send('...')```.

Remarque : Si la connexion est fermée (déclenchement de onClose), la connexion correspondante est supprimée du tableau ```$connections```.

Remarque : Les développeurs ne doivent pas modifier cette propriété, car cela pourrait entraîner des situations imprévisibles.

## Exemple

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Au démarrage du processus, définir une minuterie pour envoyer des données à toutes les connexions client à intervalles réguliers
$worker->onWorkerStart = function($worker)
{
    // Minuterie, toutes les 10 secondes
    Timer::add(10, function()use($worker)
    {
        // Parcours de toutes les connexions client du processus actuel et envoi de l'heure du serveur actuel
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Démarrer le worker
Worker::runAll();
```
