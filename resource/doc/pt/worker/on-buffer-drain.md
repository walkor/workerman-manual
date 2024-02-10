# onBufferDrain
## Descrição:
```php
callback Worker::$onBufferDrain
```

Cada conexão tem seu próprio buffer de envio de camada de aplicativo, o tamanho do buffer é determinado por ```TcpConnection::$maxSendBufferSize```, com o valor padrão de 1MB, podendo ser alterado manualmente. Após a alteração, a mudança afetará todas as conexões.

Esse retorno de chamada é acionado após todos os dados do buffer de envio de camada de aplicativo serem enviados. Geralmente, é usado em conjunto com ```onBufferFull```, por exemplo, interrompendo o envio de dados para o destinatário em ```onBufferFull``` e retomando a gravação de dados em ```onBufferDrain```.

## Parâmetros da função de retorno de chamada:
``` $connection ```

Objeto de conexão, ou seja, uma instância de [TcpConnection](../tcp-connection.md), usada para operar a conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar a conexão](../tcp-connection/close.md), etc.

## Exemplo:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull e não enviar novamente\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drenado e continuar o envio\n";
};
// Execute o worker
Worker::runAll();
```

Dica: Além de usar uma função anônima como retorno de chamada, você também pode [consultar aqui](../faq/callback_methods.md) para outras formas de retorno de chamada.

## Veja também:
onBufferFull Quando o buffer de envio de camada de aplicativo da conexão estiver cheio.
