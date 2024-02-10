# maxPackageSize

## Descrição:
```php
static int Connection::$defaultMaxPackageSize
```

Esta propriedade é uma propriedade estática global usada para definir o comprimento máximo que cada conexão pode receber. Se não for definido, o padrão é 10MB.

Se o comprimento do pacote parseado dos dados recebidos (valor retornado pelo método input da classe de protocolo) for maior do que ```Connection::$defaultMaxPackageSize```, os dados serão considerados inválidos e a conexão será encerrada.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Definir o comprimento máximo do pacote recebido por cada conexão como 1024000 bytes
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Executar o worker
Worker::runAll();
```
