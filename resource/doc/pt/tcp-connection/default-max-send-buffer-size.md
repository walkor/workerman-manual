# defaultMaxSendBufferSize
## Descrição:
```php
static int Connection::$defaultMaxSendBufferSize
```

Essa propriedade é um atributo estático global usado para definir o tamanho padrão do buffer de envio da camada de aplicação para todas as conexões. Se não for definido, o padrão é ```1MB```. ```Connection::$defaultMaxSendBufferSize``` pode ser configurado dinamicamente e terá efeito apenas nas novas conexões criadas após a configuração.

Essa propriedade afeta o retorno de chamada [onBufferFull](../worker/on-buffer-full.md).

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Definir o tamanho padrão do buffer de envio da camada de aplicação para todas as conexões
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Definir o tamanho do buffer de envio da camada de aplicação para a conexão atual, substituindo o valor padrão
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Executar o worker
Worker::runAll();
```
