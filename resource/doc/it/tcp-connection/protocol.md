# protocol

## Descrizione:
```php
string Connection::$protocol
```

Imposta la classe del protocollo per la connessione corrente.

## Esempio

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
    // Quando si invia, verrà chiamato automaticamente $connection->protocol::encode(), confezionando i dati prima di inviarli
    $connection->send("ciao");
};
// Avvia il worker
Worker::runAll();
```
