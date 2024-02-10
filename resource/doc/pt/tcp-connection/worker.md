# worker
## Descrição:
```php
Worker Connection::$worker
```

Esta propriedade é somente leitura e representa a instância do worker a que o objeto de conexão atual pertence.


## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Quando um cliente envia dados, encaminha para todos os outros clientes mantidos pelo processo atual
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Executar o worker
Worker::runAll();
```
