# Comment diffuser des données en masse

## Exemple (Diffusion planifiée)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Dans cet exemple, le nombre de processus doit être 1
$worker->count = 1;
// Lors du démarrage du processus, définissez un minuteur pour envoyer périodiquement des données à toutes les connexions client
$worker->onWorkerStart = function($worker)
{
    // Périodique, toutes les 10 secondes
    Timer::add(10, function() use ($worker)
    {
        // Parcourir toutes les connexions clients du processus actuel et envoyer l'heure actuelle du serveur
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Exécutez le worker
Worker::runAll();
```

## Exemple (Discussion de groupe)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Dans cet exemple, le nombre de processus doit être 1
$worker->count = 1;
// Lorsqu'un client envoie un message, diffusez-le à d'autres utilisateurs
$worker->onMessage = function(TcpConnection $connection, $message) use ($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Exécutez le worker
Worker::runAll();
```

## Remarque:
**Monoprocessus:**
Les exemples ci-dessus ne peuvent fonctionner qu'avec un** monoprocessus** (```$worker->count=1```), car avec plusieurs processus, plusieurs clients peuvent se connecter à des processus différents, et les clients entre les processus sont isolés et ne peuvent pas communiquer directement, c'est-à-dire qu'un processus A ne peut pas **directement** manipuler l'objet de connexion client du processus B pour envoyer des données. (Pour y parvenir, une communication inter-processus est nécessaire, par exemple, en utilisant le composant Channel, comme par exemple [exemple - envoi en cluster](../components/channel-examples.md), [exemple - envoi de groupe](../components/channel-examples2.md)).

**Il est recommandé d'utiliser GatewayWorker:**
La structure GatewayWoker développée sur la base de Workerman fournit un mécanisme de diffusion plus pratique, y compris la diffusion en groupe, la diffusion en masse, etc., et peut être configurée avec plusieurs processus, voire plusieurs déploiements de serveurs. Si vous avez besoin d'envoyer des données aux clients, il est recommandé d'utiliser la structure GatewayWorker.

Adresse du manuel GatewayWorker https://doc2.workerman.net
