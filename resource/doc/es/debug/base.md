# Depuración Básica

WorkerMan tiene dos modos de ejecución, el modo de depuración y el modo de demonio.

Al ejecutar `php start.php start`, se entra en modo de depuración, en este caso, las funciones de impresión como `echo`, `var_dump` y `var_export` en el código se mostrarán en la terminal. Es importante tener en cuenta que si se ejecuta `php start.php start`, todos los procesos de WorkerMan se cerrarán cuando se cierre la terminal.

Por otro lado, al ejecutar `php start.php start -d`, se entra en el modo demonio, es decir, el modo de ejecución oficial en línea, y no se verá afectado al cerrar la terminal.

Si se desea ver las impresiones de las funciones `echo`, `var_dump` y `var_export` incluso en el modo de ejecución de demonio, se puede configurar la propiedad `Worker::$stdoutFile`, por ejemplo:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Imprimir en pantalla la salida en el archivo especificado por Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```
De esta manera, todas las impresiones de las funciones `echo`, `var_dump` y `var_export` se escribirán en el archivo especificado por `Worker::$stdoutFile`. Es importante tener en cuenta que la ruta especificada por `Worker::$stdoutFile` debe tener permisos de escritura.
