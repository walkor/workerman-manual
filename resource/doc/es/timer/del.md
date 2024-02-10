# del
```php
boolean \Workerman\Timer::del(int $timer_id)
```
Eliminar un temporizador específico.

### Parámetros
```timer_id```

La identificación del temporizador, es decir, el entero devuelto por la interfaz de add.

### Valor de retorno
boolean


### Ejemplo
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Iniciar múltiples procesos para ejecutar tareas programadas, tenga en cuenta los problemas de concurrencia de múltiples procesos
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Ejecutar cada 2 segundos
    $timer_id = Timer::add(2, function()
    {
        echo "Tarea en ejecución\n";
    });
    // Ejecutar una tarea de una sola vez después de 20 segundos, eliminar la tarea programada de cada 2 segundos
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Ejecutar worker
Worker::runAll();
```

### Ejemplo (Eliminar el temporizador actual dentro de la llamada del temporizador)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Tenga en cuenta que para usar la identificación del temporizador actual dentro de la devolución de llamada, debe importarla por referencia (&)
    $timer_id = Timer::add(1, function() use (&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Eliminar el temporizador después de 10 ejecuciones
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Ejecutar worker
Worker::runAll();
```
