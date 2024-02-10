# onMessage
## Descrição:
```php
callback Worker::$onMessage
```
Quando os dados são recebidos pelo Workerman através da conexão do cliente, dispara a função de retorno de chamada.

## Parâmetros da função de retorno de chamada
```$connection```
Objeto de conexão, ou seja, a instância de [TcpConnection](../tcp-connection.md), utilizado para operar a conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar a conexão](../tcp-connection/close.md) etc.

```$data```
Dados enviados pela conexão do cliente, se o Worker tiver especificado um protocolo, então $data é os dados decodificados correspondentes ao protocolo. O tipo de dados depende da implementação do `decode()` do protocolo, sendo `websocket`, `text` e `frame` para strings, e o protocolo HTTP para o objeto [`Workerman\Protocols\Http\Request`](../http/request.md).


## Exemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('recebido com sucesso');
};
// Executa o worker
Worker::runAll();
```
Nota: Além de usar uma função anônima como retorno de chamada, também é [possível utilizar outros métodos de retorno de chamada](../faq/callback_methods.md).
