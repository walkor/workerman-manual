# protocol

## Descrição:
```php
string Connection::$protocol
```

Define a classe de protocolo para a conexão atual

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // Ao enviar, o $connection->protocol::encode() é chamado automaticamente para empacotar os dados antes de enviar
    $connection->send("hello");
};
// Execute o worker
Worker::runAll();
```
