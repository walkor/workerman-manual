# pauseRecv
## Description:
```php
void Connection::pauseRecv(void)
```

Arrête la réception des données pour la connexion actuelle. Le rappel onMessage de cette connexion ne sera pas déclenché. Cette méthode est très utile pour le contrôle du trafic entrant.

## Parameters
Aucun paramètre

## Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Ajoute dynamiquement une propriété à l'objet connection pour enregistrer le nombre de requêtes envoyées par la connexion actuelle
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Arrête la réception des données après réception de 100 requêtes
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Exécute le worker
Worker::runAll();
```

## Voir aussi
void Connection::resumeRecv(void) - Réactive la réception des données pour l'objet de connexion correspondant
