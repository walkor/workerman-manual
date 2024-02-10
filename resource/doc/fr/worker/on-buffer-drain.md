# onBufferDrain
## Description:
```php
callback Worker::$onBufferDrain
```

Chaque connexion a son propre tampon d'envoi de couche applicative, la taille du tampon est déterminée par ```TcpConnection::$maxSendBufferSize```, avec une valeur par défaut de 1 Mo, qui peut être modifiée manuellement, et le changement affectera toutes les connexions.

Ce rappel est déclenché lorsque toutes les données du tampon d'envoi de la couche applicative ont été complètement envoyées. Il est généralement utilisé en conjonction avec onBufferFull, par exemple, pour arrêter d'envoyer des données à la partie opposée lors de onBufferFull, et reprendre l'écriture de données lors de onBufferDrain.


## Paramètres de la fonction de rappel
 ``` $connection ```

L'objet de connexion, c'est-à-dire une instance de [TcpConnection](../tcp-connection.md), utilisée pour manipuler la connexion client, telle que [l'envoi de données](../tcp-connection/send.md), [la fermeture de la connexion](../tcp-connection/close.md), etc.


## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Tampon plein et ne pas envoyer à nouveau\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "Tampon vidé et continuer à envoyer\n";
};
// Exécute le worker
Worker::runAll();
```

Note : En plus d'utiliser une fonction anonyme comme rappel, vous pouvez également [consulter ici](../faq/callback_methods.md) pour d'autres façons d'écrire des rappels.

## Voir aussi
onBufferFull Lorsque le tampon d'envoi de la couche applicative de la connexion est plein, le rappel est déclenché
