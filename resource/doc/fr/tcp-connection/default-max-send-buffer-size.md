# defaultMaxSendBufferSize
## Description:
```php
static int Connection::$defaultMaxSendBufferSize
```

Cette propriété est une propriété statique globale utilisée pour définir la taille par défaut du tampon d'envoi de la couche applicative pour toutes les connexions. Si aucune valeur n'est définie, la valeur par défaut est de ```1 Mo```. ```Connection::$defaultMaxSendBufferSize``` peut être configurée dynamiquement, et une fois configurée, elle ne s'applique qu'aux nouvelles connexions établies ultérieurement.

Cette propriété affecte le callback [onBufferFull](../worker/on-buffer-full.md).

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Définir la taille par défaut du tampon d'envoi de la couche applicative pour toutes les connexions
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Définir la taille du tampon d'envoi de la couche applicative pour la connexion actuelle, remplaçant la valeur par défaut
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Exécuter le worker
Worker::runAll();
```
