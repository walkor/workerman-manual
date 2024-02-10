# getRemoteIp
## Descrição:
```php
string Connection::getRemoteIp()
```

Obtém o endereço IP do cliente desta conexão.

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
    echo "nova conexão do IP " . $connection->getRemoteIp() . "\n";
};
// Executar worker
Worker::runAll();
```
