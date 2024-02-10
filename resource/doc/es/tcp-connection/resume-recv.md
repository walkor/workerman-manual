# resumeRecv
## Descripción:
```php
void Connection::resumeRecv(void)
```

Permite que la conexión actual continúe recibiendo datos. Este método se utiliza en conjunto con Connection::pauseRecv y es muy útil para el control de tráfico de carga ascendente.

## Parámetros

Sin parámetros

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function (TcpConnection $connection) {
    // Agregar dinámicamente una propiedad al objeto de conexión para guardar el número de solicitudes recibidas actualmente en la conexión
    $connection->messageCount = 0;
};
$worker->onMessage = function (TcpConnection $connection, $data) {
    // Después de recibir 100 solicitudes, la conexión deja de recibir datos
    $limit = 100;
    if (++$connection->messageCount > $limit) {
        $connection->pauseRecv();
        // Se reanudará la recepción de datos después de 30 segundos
        Timer::add(30, function ($connection) {
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Ejecutar el worker
Worker::runAll();
```

## Ver también
void Connection::pauseRecv(void) detiene la recepción de datos del objeto de conexión correspondiente
