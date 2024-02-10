# onError
## Description:
```php
callback Worker::$onError
```

Déclenché lorsqu'une erreur se produit sur la connexion du client.

Actuellement, les types d'erreurs sont les suivants :

1. Échec de l'appel à Connection::send en raison de la déconnexion du client (suivi immédiatement par le déclenchement de la fonction de rappel onClose) ``` (code: WORKERMAN_SEND_FAIL msg: client closed) ```

2. Après le déclenchement de onBufferFull (le tampon d'envoi est plein), un nouvel appel à Connection::send est effectué, mais le tampon d'envoi est toujours plein, ce qui entraîne un échec de l'envoi (la fonction de rappel onClose n'est pas déclenchée) ``` (code: WORKERMAN_SEND_FAIL msg: send buffer full and drop package) ```

3. Échec de la connexion asynchrone avec AsyncTcpConnection (suivi immédiatement par le déclenchement de la fonction de rappel onClose) ``` (code: WORKERMAN_CONNECT_FAIL msg: message d'erreur renvoyé par stream_socket_client) ```


## Callback Function Parameters

``` $connection ```

Objet de connexion, c'est-à-dire une instance de TcpConnection utilisée pour manipuler la connexion du client, telle que [l'envoi de données](../tcp-connection/send.md), [la fermeture de la connexion](../tcp-connection/close.md), etc.

``` $code ```

Code d'erreur

``` $msg ```

Message d'erreur


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// Exécuter le worker
Worker::runAll();
```

Remarque : En plus d'utiliser une fonction anonyme comme rappel, vous pouvez également [consulter ici](../faq/callback_methods.md) d'autres façons d'écrire des rappels.
