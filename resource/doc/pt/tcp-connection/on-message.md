# onMessage
## Explicação
```php
callback Connection::$onMessage
```

Tem o mesmo efeito que o retorno de [Worker::$onMessage](../worker/on-message.md), mas a diferença é que só é válido para a conexão atual, ou seja, pode ser configurado um retorno onMessage específico para uma determinada conexão.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Quando ocorrer um evento de conexão com o cliente
$worker->onConnect = function(TcpConnection $connection)
{
    // Configurar o retorno onMessage da conexão
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('recebido com sucesso');
    };
};
// Executar o worker
Worker::runAll();
```

O código acima tem o mesmo efeito que o seguinte:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Configurar diretamente o retorno onMessage para todas as conexões
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('recebido com sucesso');
};
// Executar o worker
Worker::runAll();
```
