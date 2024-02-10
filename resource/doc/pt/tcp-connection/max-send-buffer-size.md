# maxSendBufferSize
## Descrição:
```php
int Connection::$maxSendBufferSize
```

Cada conexão possui um buffer de envio de camada de aplicativo separado. Se a velocidade de recebimento do cliente for menor do que a velocidade de envio do servidor, os dados serão temporariamente armazenados no buffer de camada de aplicativo aguardando envio.

Esta propriedade é usada para definir o tamanho do buffer de envio de camada de aplicativo da conexão atual. Se não for definido, o padrão é [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1 MB).

Esta propriedade afeta o retorno de chamada [onBufferFull](../worker/on-buffer-full.md).

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Define o tamanho do buffer de envio de camada de aplicativo da conexão atual como 102400 bytes
    $connection->maxSendBufferSize = 102400;
};
// Executa o worker
Worker::runAll();
```
