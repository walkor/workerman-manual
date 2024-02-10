# onClose
## Descrição
```php
callback Connection::$onClose
```

Este retorno de chamada é o mesmo que o retorno de chamada [Worker::$onClose](../worker/on-close.md), a diferença é que ele só é válido para a conexão atual, ou seja, pode ser definido um retorno de chamada onClose para uma conexão específica.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Quando há um evento de conexão, aciona
$worker->onConnect = function(TcpConnection $connection)
{
    // Define o retorno de chamada onClose para a conexão
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "conexão fechada\n";
    };
};
// Execute o worker
Worker::runAll();
```

O código acima é equivalente ao seguinte:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Define o retorno de chamada onClose para todas as conexões
$worker->onClose = function(TcpConnection $connection)
{
    echo "conexão fechada\n";
};
// Execute o worker
Worker::runAll();
```
