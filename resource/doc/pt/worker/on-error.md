# onError
## Descrição:
```php
callback Worker::$onError
```
Disparado quando ocorre um erro na conexão do cliente.

Atualmente, os tipos de erros incluem

1. Falha ao chamar Connection::send devido a desconexão do cliente (em seguida, o callback onClose será disparado) ``` (code:WORKERMAN_SEND_FAIL msg:client closed) ```

2. Após o disparo de onBufferFull (quando o buffer de envio está cheio), ainda ocorre a chamada para Connection::send e o buffer de envio permanece cheio, resultando em falha no envio (o callback onClose não será disparado) ``` (code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package) ```

3. Falha na conexão assíncrona do AsyncTcpConnection (em seguida, o callback onClose será disparado) ``` (code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client retorna a mensagem de erro) ```

## Parâmetros da função de retorno

 ``` $connection ```

Objeto de conexão, ou seja, a instância TcpConnection, usado para operar a conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar a conexão](../tcp-connection/close.md), etc.

 ``` $code ```

Código de erro

 ``` $msg ```

Mensagem de erro

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// Execute o worker
Worker::runAll();
```

Dica: Além de usar uma função anônima como retorno, você também pode [consultar aqui](../faq/callback_methods.md) outros métodos de escrita de retorno.
