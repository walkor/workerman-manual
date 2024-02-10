# onMessage
## Description:
```php
callback Worker::$onMessage
```

La fonction de rappel déclenchée lorsque les données sont reçues par Workerman lorsque le client envoie des données via une connexion.

## Paramètres de la fonction de rappel

 ``` $connection ```

L'objet de connexion, c'est-à-dire l'instance [TcpConnection](../tcp-connection.md), utilisée pour manipuler la connexion client, telle que [envoyer des données](../tcp-connection/send.md), [fermer la connexion](../tcp-connection/close.md), etc.

 ``` $data ```

Les données envoyées par la connexion client. Si le Worker spécifie un protocole, alors $data est les données décodées correspondantes au protocole. Le type de données dépend de l'implémentation de la méthode `decode()` du protocole. Pour les protocoles `websocket`, `text` et `frame`, il s'agit d'une chaîne de caractères, pour le protocole HTTP, il s'agit d'un objet [`Workerman\Protocols\Http\Request`](../http/request.md).

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Exécuter le worker
Worker::runAll();
```

Remarque : En plus de l'utilisation de fonctions anonymes comme rappel, vous pouvez également [voir ici](../faq/callback_methods.md) pour d'autres façons d'écrire des rappels.
