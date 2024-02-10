# protocol

## Descripción:
```php
string Connection::$protocol
```

Establece la clase de protocolo para la conexión actual.

## Ejemplo

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
    // Al enviar, se llamará automáticamente a $connection->protocol::encode() para empaquetar los datos antes de enviarlos
    $connection->send("hello");
};
// Ejecutar el worker
Worker::runAll();
```
