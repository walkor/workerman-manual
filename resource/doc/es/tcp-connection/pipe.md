## pipe
## Descripción:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Parámetros
Dirige el flujo de datos de la conexión actual a la conexión de destino. Incluye control de flujo. Este método es muy útil para hacer de proxy TCP.

## Ejemplo de proxy TCP
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// Después de que se establezca la conexión TCP
$worker->onConnect = function(TcpConnection $connection)
{
    // Establecer una conexión asincrónica al puerto 80 local
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Establecer la dirección de los datos de la conexión del cliente actual al puerto 80 de la conexión
    $connection->pipe($connection_to_80);
    // Establecer la dirección de los datos devueltos por la conexión al puerto 80 al cliente
    $connection_to_80->pipe($connection);
    // Conectar de forma asincrónica
    $connection_to_80->connect();
};

// Ejecutar el worker
Worker::runAll();
```
