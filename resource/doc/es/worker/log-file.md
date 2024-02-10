# logFile
## Descripción:
```php
static string Worker::$logFile
```

Se utiliza para especificar la ubicación del archivo de registro de Workerman.

Este archivo registra los registros relacionados con Workerman en sí, como el inicio, la parada, etc.

Si no se ha configurado, el nombre de archivo predeterminado es workerman.log y la ubicación del archivo está en el directorio padre de Workerman.

**Nota:**

Este archivo de registro solo registra los registros relacionados con el inicio y la parada de Workerman, no incluye ningún registro comercial.

Las clases de registro comerciales pueden implementarse utilizando funciones como [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) o [error_log](https://php.net/manual/zh/function.error-log.php).

## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Inicio del worker";
};
// Ejecutar el worker
Worker::runAll();
```
