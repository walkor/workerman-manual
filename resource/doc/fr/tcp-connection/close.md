# close
## Description:
```php
void Connection::close(mixed $data = '')
```

Ferme la connexion de manière sécurisée.

Appeler `close` attend que les données du tampon d'envoi soient envoyées avant de fermer la connexion, puis déclenche le rappel `onClose` de la connexion.

## Arguments

``` $data ```

Paramètre facultatif, les données à envoyer (si un protocole est spécifié, la méthode d'encodage du protocole est automatiquement appelée pour empaqueter les données de ```$data```), une fois les données envoyées, la connexion est fermée, puis le rappel `onClose` est déclenché.

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Exécute le worker
Worker::runAll();
```
