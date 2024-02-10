# pauseRecv
## Descrição:
```php
void Connection::pauseRecv(void)
```

Interrompe a recepção de dados da conexão atual. O retorno de chamada onMessage desta conexão não será acionado. Este método é muito útil para controlar o tráfego de upload.

## Parâmetros

Sem parâmetros

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Adiciona dinamicamente um atributo ao objeto de conexão para receber e armazenar o número de solicitações recebidas pela conexão atual
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // A conexão para de receber dados após 100 solicitações recebidas
    $limite = 100;
    if(++$connection->messageCount > $limite)
    {
        $connection->pauseRecv();
    }
};
// Inicia o worker
Worker::runAll();
```

## Veja também
void Connection::resumeRecv(void) - Faz com que o objeto de conexão correspondente retome a recepção de dados
