# getRemotePort
## Descrição:
```php
int Connection::getRemotePort()
```

Obtém a porta do cliente desta conexão.

## Parâmetros

Sem parâmetros

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nova conexão do endereço " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Execute o worker
Worker::runAll();
```
