# reloadable
## Descripción:
```php
bool Worker::$reloadable
```

Cuando se ejecuta `php start.php reload`, se enviará una señal de recarga (SIGUSR1) a todos los procesos secundarios.

Una vez que un proceso secundario recibe la señal de recarga, se cerrará automáticamente y el proceso principal lanzará automáticamente un nuevo proceso, generalmente utilizado para actualizar el código comercial.

Cuando el proceso $reloadable es false, al recibir la señal de recarga, solo se activará [onWorkerReload](on-worker-reload.md), y no reiniciará el proceso actual.

Por ejemplo, en el modelo de puerta de enlace/worker, el proceso de la puerta de enlace es responsable de mantener las conexiones de los clientes y el proceso del trabajador es responsable de manejar las solicitudes. Al configurar la propiedad reloadable de la puerta de enlace como false, se puede actualizar el código comercial sin desconectar las conexiones de los clientes durante la recarga.

## Ejemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establecer si este ejemplo se reinicia después de recibir la señal de recarga
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Comenzando el trabajador...\n";
};
// Ejecutar el trabajador
Worker::runAll();
```
