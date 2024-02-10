# onConnect
## Descrição:
```php
callback Worker::$onConnect
```

Quando o cliente estabelece uma conexão com o Workerman (após a conclusão do handshake TCP), é acionada a função de retorno de chamada ```onConnect```. Cada conexão aciona a função de retorno de chamada ```onConnect``` apenas uma vez.

Nota: O evento onConnect indica apenas que o cliente completou o handshake TCP com o Workerman. Neste momento, o cliente ainda não enviou nenhum dado. Além de obter o IP do cliente através de ```$connection->getRemoteIp()```, não há outros dados ou informações para identificar o cliente neste evento onConnect. Portanto, não é possível identificar quem é o cliente no evento onConnect. Para saber quem é o cliente, é necessário que o cliente envie dados de autenticação, como um token ou nome de usuário e senha, e fazer a autenticação na [função de retorno onMessage](on-message.md).

Como o UDP é sem conexão, quando usado o UDP, o evento onConnect não será acionado e também não acionará o evento onClose.

## Parâmetros da função de retorno de chamada

 ``` $connection ```

Objeto de conexão, ou seja, a instância [TcpConnection](../tcp-connection.md), usado para operar a conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar a conexão](../tcp-connection/close.md), etc.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nova conexão do IP " . $connection->getRemoteIp() . "\n";
};
// Executar o worker
Worker::runAll();
```

Dica: Além de usar funções anônimas como retorno de chamada, também é possível [consultar aqui](../faq/callback_methods.md) outras formas de escrever retorno de chamada.
