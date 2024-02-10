# protocole

## Description:
```php
string Connection::$protocol
```

Définit la classe de protocole actuelle pour la connexion.


## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // Lors de l'envoi, $connection->protocol::encode() est automatiquement appelé pour empaqueter les données avant l'envoi
    $connection->send("hello");
};
// Exécutez le worker
Worker::runAll();
```
