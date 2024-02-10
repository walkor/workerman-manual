```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Ejecuta una función o método de una clase de forma programada.

Nota: Los temporizadores se ejecutan en el proceso actual, workerman no creará nuevos procesos o hilos para ejecutar el temporizador.

### Parámetros
 ``` time_interval ```

Intervalo de tiempo en segundos para la ejecución, admite decimales y puede ser preciso hasta 0.001, es decir, a nivel de milisegundos.

 ``` callback ```

La función de devolución de llamada. **Nota: Si la función de devolución de llamada es un método de una clase, el método debe ser de acceso público.**

 ``` args ```

Los argumentos de la función de devolución de llamada, deben ser un array con los valores de los parámetros.

 ``` persistent ```

Indica si es persistente. Si desea que la tarea se ejecute solo una vez, pase false (las tareas que se ejecutan solo una vez se destruirán automáticamente después de su ejecución, no es necesario llamar a ```Timer::del()```). El valor predeterminado es true, es decir, se ejecutará continuamente. 

### Valor de retorno
Devuelve un entero que representa el timerid del temporizador, el cual se puede destruir llamando a ```Timer::del($timerid)```.

### Ejemplos

#### 1. Función programada como función anónima (cierre)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "Tarea ejecutada\n";
    });
};

// Ejecutar worker
Worker::runAll();
```
