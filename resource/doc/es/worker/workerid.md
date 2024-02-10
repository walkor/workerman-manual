# id

Requires ```（workerman >= 3.2.1）```

## Descripción:
```php
int Worker::$id
```

El número de identificación del proceso actual del worker, que varía de ```0``` a ```$worker->count-1```.

Esta propiedad es muy útil para diferenciar los procesos del worker. Por ejemplo, si una instancia del worker tiene múltiples procesos y el desarrollador solo quiere establecer un temporizador en uno de los procesos, puede lograrlo identificando el número de identificación del proceso y estableciendo el temporizador solo en el proceso con la identificación 0 (ver ejemplo).

## Nota:

El valor del número de identificación del proceso no cambia después de reiniciar el proceso.

La asignación del número de identificación del proceso se basa en cada instancia del worker. Cada instancia del worker comienza asignando el número de identificación del proceso desde 0, por lo que puede haber identificaciones de procesos duplicadas entre las instancias del worker, pero no habrá duplicados dentro de una misma instancia del worker. Por ejemplo, en el siguiente ejemplo:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// La instancia del worker 1 tiene 4 procesos, y los números de identificación de los procesos serán 0, 1, 2, 3 respectivamente.
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Establecer 4 procesos activos
$worker1->count = 4;
// Imprimir el número de identificación del proceso actual después de iniciar cada proceso
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// La instancia del worker 2 tiene 2 procesos, y los números de identificación de los procesos serán 0 y 1 respectivamente.
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Establecer 2 procesos activos
$worker2->count = 2;
// Imprimir el número de identificación del proceso actual después de iniciar cada proceso
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Ejecutar el worker
Worker::runAll();
```
La salida será similar a:
``` 
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Nota: Debido a que Windows no admite la configuración del número de procesos (count), solo habrá un proceso con el número de identificación 0 en el sistema Windows. Y en sistemas Windows no se admite la inicialización de dos instancias del Worker para escucha de eventos. Por lo tanto, este ejemplo no se puede ejecutar en sistemas Windows.

## Ejemplo
Una instancia del worker tiene 4 procesos; solo se establece un temporizador en el proceso con el número de identificación 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Establecer un temporizador solo en el proceso con el número de identificación 0; los procesos 1, 2 y 3 no tendrán temporizadores configurados.
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 procesos del worker; solo se establece un temporizador en el proceso 0\n";
        });
    }
};
// Ejecutar el worker
Worker::runAll();
```
