# resumeRecv
## Description:
```php
void Connection::resumeRecv(void)
```

Permet à la connexion actuelle de continuer à recevoir des données. Cette méthode est utile en combinaison avec Connection::pauseRecv pour contrôler efficacement le trafic montant.

## Parameters
Aucun paramètre

## Example
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Ajoute dynamiquement une propriété à l'objet connection pour enregistrer le nombre de requêtes reçues par la connexion actuelle
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // La connexion ne recevra plus de données après avoir reçu 100 requêtes
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Résume la réception des données après 30 secondes
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Exécute le worker
Worker::runAll();
```

## See also
void Connection::pauseRecv(void) arrête la réception de données pour l'objet de connexion correspondant
