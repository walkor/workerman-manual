# pauseRecv
## Descripción:
```php
void Connection::pauseRecv(void)
```
Detiene la recepción de datos en la conexión actual. El callback onMessage de esta conexión no será activado. Este método es útil para el control de tráfico de carga ascendente.

## Parámetros
Sin parámetros

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Añadir dinámicamente una propiedad al objeto de conexión para guardar cuántas solicitudes ha enviado la conexión actual
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // La conexión deja de recibir datos después de 100 solicitudes
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Ejecutar el worker
Worker::runAll();
```

## Ver también
void Connection::resumeRecv(void) - Reanuda la recepción de datos en el objeto de conexión correspondiente
