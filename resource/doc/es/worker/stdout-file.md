# stdoutFile
## Descripción:
```php
static string Worker::$stdoutFile
```
Este atributo es un atributo estático global. Si se inicia en modo daemon (usando ```-d```), toda la salida hacia la terminal (echo, var_dump, etc.) será redirigida al archivo especificado en stdoutFile.

Si no se establece y se ejecuta en modo daemon, toda la salida de la terminal se redirige a `/dev/null` (es decir, se descarta toda la salida de forma predeterminada).

> Nota: `/dev/null` es un archivo especial en Linux que es en realidad un agujero negro, todos los datos escritos en este archivo se descartarán. Si no desea descartar la salida, puede configurar `Worker::$stdoutFile` como una ruta de archivo normal.

> Nota: Este atributo debe establecerse antes de que se ejecute ```Worker::runAll();```. Este atributo no es compatible con el sistema Windows.

## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Toda la salida impresa se guarda en el archivo /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Inicio del worker\n";
};
// Ejecutar el worker
Worker::runAll();
```
