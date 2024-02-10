# destroy
## Descrição:
```php
void Connection::destroy()
```

Fecha imediatamente a conexão.

A diferença entre destroy e close é que ao chamar destroy, mesmo que haja dados não enviados para o endpoint no buffer de envio da conexão, a conexão será fechada imediatamente e o callback ```onClose``` será imediatamente acionado.

## Parâmetros

Sem parâmetros

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // se algo estiver errado
    $connection->destroy();
};
// execute o worker
Worker::runAll();
```
