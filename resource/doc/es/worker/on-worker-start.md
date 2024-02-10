# onWorkerStart
## Descripción:
```php
callback Worker::$onWorkerStart
```
Establece la función de devolución de llamada al iniciar el subproceso del Worker, que se ejecuta cada vez que se inicia un subproceso del Worker.

Nota: onWorkerStart se ejecuta cuando se inicia el subproceso, y si se inician múltiples subprocesos (```$worker->count > 1```), se ejecutará ```$worker->count``` veces en total.


## Parámetros de la función de devolución de llamada

 ``` $worker ```

Se refiere al objeto Worker.


## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Iniciando el Worker...\n";
};
// Ejecutar el worker
Worker::runAll();
```

Nota: Además de usar funciones anónimas como devolución de llamada, también se puede [consultar aquí](../faq/callback_methods.md) para otras formas de escritura de devoluciones de llamada.
