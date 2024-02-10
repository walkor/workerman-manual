# maxSendBufferSize
## Description:
```php
int Connection::$maxSendBufferSize
```

Chaque connexion possède un tampon d'envoi de la couche applicative, si la vitesse de réception du client est inférieure à la vitesse d'envoi du serveur, les données seront mises en attente dans le tampon de la couche applicative en attendant d'être envoyées.

Cette propriété est utilisée pour définir la taille du tampon d'envoi de la couche applicative de la connexion actuelle. Si non définie, la taille par défaut est [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)(1 Mo).

Cette propriété affecte le rappel [onBufferFull](../worker/on-buffer-full.md).

## Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Définir la taille du tampon d'envoi de la couche applicative de la connexion actuelle à 102400 octets
    $connection->maxSendBufferSize = 102400;
};
// Exécuter le worker
Worker::runAll();
```
