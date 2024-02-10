# id

## Descripción
```php
int Connection::$id
```
El id de la conexión. Este es un entero que se incrementa automáticamente.

Nota: Workerman es multiproceso, cada proceso mantiene un id de conexión que se incrementa automáticamente, por lo que puede haber duplicados de id de conexión entre varios procesos. Si se desea evitar duplicados de id de conexión, se puede reasignar el valor de `$connection->id` según sea necesario, por ejemplo, agregando un prefijo `worker->id`.

## Ver también
[Propiedad connections de Worker](../worker/connections.md)

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Ejecutar el worker
Worker::runAll();
```
