# onWorkerReload
Requerido ```(workerman >= 3.2.5)```
## Descripción:
```php
callback Worker::$onWorkerReload
```
Esta característica no se usa con frecuencia.

Establece el callback que se ejecutará cuando el Worker reciba la señal de recarga.

Se puede utilizar el callback onWorkerReload para realizar muchas tareas, como recargar archivos de configuración de negocios sin necesidad de reiniciar el proceso.

**Nota**:

La acción predeterminada para un subproceso al recibir la señal de recarga es salir y reiniciar, para que el nuevo proceso cargue el código del negocio y complete la actualización del código. Por lo tanto, es normal que el subproceso salga inmediatamente después de ejecutar el callback onWorkerReload.

Si desea que el subproceso solo ejecute onWorkerReload después de recibir la señal de recarga y no salga, puede establecer la propiedad "reloadable" de la instancia correspondiente de Worker en false al inicializar la instancia de Worker.


## Parámetros de la función de callback
 ``` $worker ```

Es el objeto Worker.

## Ejemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establece reloadable en false, es decir, el subproceso no ejecuta reinicio al recibir la señal de recarga
$worker->reloadable = false;
// Después de la recarga, informa a todos los clientes que el servidor ha recargado
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Ejecuta el worker
Worker::runAll();
```

Nota: Además de usar una función anónima como callback, también se puede [consultar aquí](../faq/callback_methods.md) otros métodos de escritura de callback.
