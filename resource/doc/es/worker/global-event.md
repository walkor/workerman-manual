# globalEvent

## Descripción:
```php
static Event Worker::$globalEvent
```

Este atributo es un atributo estático global que representa una instancia global del bucle de evento, y se puede usar para registrar eventos de lectura/escritura de descriptores de archivo o eventos de señales.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'El PID es ' . posix_getpid() . "\n";
    // Cuando el proceso recibe la señal SIGALRM, imprime algunas informaciones
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Recibir la señal SIGALRM\n";
    });
};
// Ejecutar el worker
Worker::runAll();
```

## Prueba
Cuando Workerman se inicia, se imprimirá el PID actual del proceso (un número). Ejecuta lo siguiente en la línea de comandos:
```bash
kill -SIGALRM pid_del_proceso
```
El servidor imprimirá:
```bash
Recibir la señal SIGALRM
```
