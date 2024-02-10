# id

## Descrição:
```php
int Connection::$id
```

O id da conexão. Este é um número inteiro incremental.

Nota: Workerman é multiprocessado, cada processo mantém um id de conexão incremental, então pode haver duplicação de ids de conexão entre vários processos. Se desejar ids de conexão não duplicados, eles podem ser reatribuídos conforme necessário, por exemplo, adicionando um prefixo com worker->id.

## Veja também
[Propriedade connections do Worker](../worker/connections.md)

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Executar worker
Worker::runAll();
```
