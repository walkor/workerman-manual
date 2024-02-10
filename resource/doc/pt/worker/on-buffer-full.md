# onBufferFull
## Descrição:
```php
callback Worker::$onBufferFull
```

Cada conexão possui um buffer de envio de camada de aplicação separado. Se a velocidade de recebimento do cliente for menor do que a velocidade de envio do servidor, os dados serão temporariamente armazenados no buffer de camada de aplicação e, se o buffer estiver cheio, o callback onBufferFull será acionado.

O tamanho do buffer é definido por [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), com valor padrão de 1 MB. O tamanho do buffer pode ser configurado dinamicamente para a conexão atual, por exemplo:
```php
// Definir o tamanho do buffer de envio para a conexão atual, em bytes
$connection->maxSendBufferSize = 102400;
```
Também é possível usar [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) para definir o tamanho padrão do buffer para todas as conexões, como no exemplo a seguir:
```php
use Workerman\Connection\TcpConnection;
// Definir o tamanho padrão do buffer de envio de camada de aplicação para todas as conexões, em bytes
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

Este callback **pode** ser acionado imediatamente após a chamada Connection::send, como no caso de envio de grandes quantidades de dados ou envio rápido e contínuo de dados para o outro lado devido a problemas de rede, resultando em uma grande acumulação de dados no buffer de envio da conexão correspondente, acionando o callback quando excede o limite de ```TcpConnection::$maxSendBufferSize```.

Quando ocorre o evento onBufferFull, geralmente é necessário que o desenvolvedor tome medidas, como parar de enviar dados para o outro lado e aguardar a conclusão do envio dos dados no buffer (evento onBufferDrain), entre outras ações.

Quando a chamada Connection::send(`$A`) aciona o onBufferFull, independentemente do tamanho dos dados `$A` enviados, mesmo que exceda o `TcpConnection::$maxSendBufferSize`, os dados a serem enviados ainda serão armazenados no buffer de envio. Em outras palavras, o tamanho real do buffer de envio pode ser muito maior que `TcpConnection::$maxSendBufferSize`. Quando o buffer de envio já contém mais dados do que `TcpConnection::$maxSendBufferSize`, se a operação Connection::send(`$B`) for executada, os dados `$B` desta operação serão descartados e acionarão o callback `onError`.

Resumindo, enquanto o buffer de envio não estiver cheio, mesmo que haja apenas um byte de espaço, a chamada Connection::send(```$A```) certamente irá armazenar o `$A` no buffer de envio. Se, após o armazenamento no buffer de envio, o tamanho do buffer ultrapassar o limite de `TcpConnection::$maxSendBufferSize`, o callback onBufferFull será acionado.

## Parâmetros da função de retorno

 ``` $connection ```

O objeto de conexão, ou seja, a instância [TcpConnection](../tcp-connection.md), usado para operar a conexão do cliente, como [enviar dados](../tcp-connection/send.md), [fechar a conexão](../tcp-connection/close.md), entre outras operações.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Buffer cheio e não enviar novamente\n";
};
// Executar o worker
Worker::runAll();
```

Nota: Além de usar uma função anônima como callback, também é possível usar [outros métodos de callback](../faq/callback_methods.md) como referência.

## Veja também
onBufferDrain Quando todos os dados no buffer de envio da camada de aplicação da conexão forem enviados completamente
