# pipe
## Descrição:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Parâmetros
Direciona o fluxo de dados da conexão atual para a conexão de destino. O controle de tráfego está integrado. Este método é muito útil para a criação de proxies TCP.

## Exemplo de proxy TCP

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// Após a conexão TCP ser estabelecida
$worker->onConnect = function(TcpConnection $connection)
{
    // Estabelece uma conexão assíncrona com a porta 80 local
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Define a direção dos dados da conexão do cliente atual para a conexão da porta 80
    $connection->pipe($connection_to_80);
    // Define a direção dos dados retornados pela conexão da porta 80 para a conexão do cliente
    $connection_to_80->pipe($connection);
    // Estabelece a conexão assíncrona
    $connection_to_80->connect();
};

// Executa o worker
Worker::runAll();
```
