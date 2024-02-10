# onClose
## Descrição:
```php
callback Worker::$onClose
```

Callback function triggered when a client connection disconnects from Workerman. Regardless of how the connection is disconnected, the `onClose` function will be triggered. Each connection will only trigger `onClose` once.

Note: If the opposite end disconnects due to extreme situations such as network disconnection or power outage, Workerman will not be able to timely send a TCP fin packet to learn that the connection has been disconnected, and thus will not be able to promptly trigger `onClose`. This situation needs to be resolved through application-layer heartbeats. An example of connection heartbeats in Workerman can be found [here](../faq/heartbeat.md). If using the GatewayWorker framework, simply use the GatewayWorker framework's heartbeat mechanism, as seen [here](https://doc2.workerman.net/heartbeat.html).

Because UDP is connectionless, when using UDP, the onConnect callback will not be triggered, nor will the onClose callback.

## Parâmetros da função de retorno

 ``` $connection ```

Objeto de conexão, ou seja, a instância [TcpConnection](../tcp-connection.md), utilizada para operações na conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar conexão](../tcp-connection/close.md), entre outros.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "conexão fechada\n";
};
// Executar o worker
Worker::runAll();
```

Nota: Além de usar uma função anônima como retorno, também é possível [consultar aqui](../faq/callback_methods.md) outras formas de escrita para retornos de chamada.
