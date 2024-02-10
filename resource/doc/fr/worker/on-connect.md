# onConnect
## Description:
```php
callback Worker::$onConnect
```

Le callback ```$onConnect``` de la classe Worker est déclenché lorsque le client établit une connexion avec Workerman (après l'achèvement du handshake TCP). Le callback ```onConnect``` ne sera déclenché qu'une seule fois pour chaque connexion.

Remarque : l'événement onConnect ne signifie que le client a établi le handshake TCP avec Workerman, à ce stade, le client n'a pas encore envoyé de données. A part l'obtention de l'adresse IP distante via ```$connection->getRemoteIp()```, il n'y a pas d'autres données ou informations permettant d'identifier le client. Par conséquent, il n'est pas possible de savoir qui est le client dans l'événement onConnect. Pour savoir qui est le client, il est nécessaire que le client envoie des données d'authentification, telles qu'un jeton (token) ou un nom d'utilisateur et un mot de passe, et faire l'authentification dans le [callback onMessage](on-message.md).

Comme l'UDP est sans connexion, lorsque l'UDP est utilisé, le callback onConnect ne sera pas déclenché, de même que le callback onClose.

## Paramètres du callback
 ``` $connection ```

L'objet de la connexion, c'est-à-dire une instance de [TcpConnection](../tcp-connection.md), utilisé pour opérer la connexion client, par exemple [envoyer des données](../tcp-connection/send.md), [fermer la connexion](../tcp-connection/close.md), etc.

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nouvelle connexion depuis l'adresse IP " . $connection->getRemoteIp() . "\n";
};
// Exécution du worker
Worker::runAll();
```

Note : En plus de l'utilisation de la fonction anonyme comme callback, il est également possible de consulter [ce lien](../faq/callback_methods.md) pour d'autres façons d'écrire des callbacks.
