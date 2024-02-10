# maxPackageSize

## Description:
```php
static int Connection::$defaultMaxPackageSize
```

Cette propriété est une propriété statique globale utilisée pour définir la longueur maximale de chaque paquet que chaque connexion peut recevoir. Par défaut, elle est de 10 Mo.

Si la longueur du paquet obtenue lors de l'analyse du paquet envoyé (valeur de retour de la méthode d'entrée de la classe de protocole) est supérieure à ```Connection::$defaultMaxPackageSize```, le paquet sera considéré comme invalide et la connexion sera interrompue.


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Définit la taille maximale du paquet à recevoir pour chaque connexion à 1024000 octets
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Exécuter le worker
Worker::runAll();
```
