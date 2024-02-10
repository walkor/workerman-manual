# resumeRecv
## Descrição:
```php
void Connection::resumeRecv(void)
```

Continua a receber os dados da conexão atual. Este método é usado em conjunto com Connection::pauseRecv e é muito útil para controlar o tráfego de upload.

## Parâmetros
Sem parâmetros

## Exemplo
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Adiciona dinamicamente um atributo ao objeto de conexão para salvar quantas solicitações foram recebidas pela conexão atual
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Após receber 100 solicitações, a conexão não aceitará mais dados
    $limite = 100;
    if(++$connection->messageCount > $limite)
    {
        $connection->pauseRecv();
        // Resume a recepção de dados após 30 segundos
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Executa o worker
Worker::runAll();
```

## Veja também
void Connection::pauseRecv(void) - Para a conexão correspondente de receber dados
