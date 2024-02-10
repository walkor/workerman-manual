# destroy
## Description:
```php
void Connection::destroy()
```

Ferme immédiatement la connexion.

Contrairement à close, l'appel à destroy entraîne la fermeture immédiate de la connexion, même si le tampon d'envoi de cette connexion contient encore des données non envoyées à la partie distante, et déclenche immédiatement le rappel ```onClose``` de cette connexion.

## Parameters

No parameters

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// Run worker
Worker::runAll();
```
