# getRemoteIp
## Description:
```php
string Connection::getRemoteIp()
```

Obtient l'adresse IP du client de cette connexion.

## Parameters
No parameters

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nouvelle connexion depuis l'adresse IP " . $connection->getRemoteIp() . "\n";
};
// Exécution du worker
Worker::runAll();
```
