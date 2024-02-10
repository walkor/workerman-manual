# maxPackageSize

## Descripción:
```php
static int Connection::$defaultMaxPackageSize
```

Esta propiedad es una propiedad estática global que se utiliza para establecer la longitud máxima de un paquete que cada conexión puede recibir. Si no se establece, el valor predeterminado es de 10 MB.

Si el paquete de datos recibido después de analizarlo (valor devuelto por el método de entrada de la clase del protocolo) tiene una longitud mayor que ```Connection::$defaultMaxPackageSize```, se considerará como datos no válidos y se desconectará la conexión.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Establecer la longitud máxima de los paquetes de datos recibidos para cada conexión en 1024000 bytes
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Ejecutar el worker
Worker::runAll();
```
