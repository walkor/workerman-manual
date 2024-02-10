# close
## Descrição:
```php
void Connection::close(mixed $data = '')
```

Fecha a conexão de forma segura.

Chamar o método close aguardará o envio completo dos dados no buffer de envio antes de fechar a conexão e acionar o callback ```onClose```.

## Parâmetros

 ``` $data ```

Parâmetro opcional, os dados a serem enviados (se um protocolo específico estiver especificado, o método encode do protocolo será automaticamente chamado para empacotar os dados ```$data```), após o envio dos dados, a conexão será fechada e em seguida o callback onClose será acionado.


## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Executar o worker
Worker::runAll();
```
