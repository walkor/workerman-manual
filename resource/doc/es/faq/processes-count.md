# ¿Cuántos procesos deberían ser abiertos?

## Cómo configurar el número de procesos
El número de procesos está determinado por la propiedad ```count``` (no compatible con sistemas Windows), como se muestra en el siguiente código:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Iniciar 4 procesos para el servicio ##
$http_worker->count = 4;

...
```

## Consideraciones para configurar el número de procesos
1. Número de núcleos de CPU.
2. Tamaño de la memoria.
3. Si la carga de trabajo es intensiva en E/S o en la CPU.

## Principios para configurar el número de procesos
1. La suma de la memoria utilizada por cada proceso debe ser menor que la memoria total (por lo general, cada proceso empresarial utiliza alrededor de 40 MB de memoria).
2. Si es intensivo en E/S, lo que significa que involucra algunas operaciones de E/S **bloqueantes**, como acceder a bases de datos Mysql, Redis, etc., se puede aumentar el número de procesos, por ejemplo, configurándolos en 3 veces el número de núcleos de CPU. Si hay muchas operaciones de bloqueo en espera en el negocio, el número de procesos también se puede aumentar adecuadamente, por ejemplo, hasta 8 veces el número de núcleos de CPU o incluso más. Tenga en cuenta que las operaciones de E/S **no bloqueantes** entran en la categoría de intensivo en la CPU y no en la de intensivo en E/S.
3. Si es intensivo en la CPU, lo que significa que no hay costos **bloqueantes** de E/S en el negocio, por ejemplo, usar E/S asincrónica para leer recursos de red, y los procesos no se bloquean debido al código empresarial, entonces el número de procesos puede configurarse igual que el número de núcleos de CPU.

## Valores de referencia para configurar el número de procesos
Si el código empresarial es intensivo en E/S, configure el número de procesos según el grado de intensidad en E/S, como de 3 a 8 veces el número de núcleos de CPU.

Si el código empresarial es intensivo en la CPU, el número de procesos puede establecerse igual que el número de núcleos de CPU.

## Nota
La E/S en Workerman en sí misma es no bloqueante, por ejemplo, ```Connection->send```, todas son operaciones no bloqueantes, que pertenecen a la categoría de intensivo en la CPU. Si no está seguro de qué tipo de carga de trabajo tiene su negocio, puede configurar el número de procesos en aproximadamente 3 veces el número de núcleos de CPU.
Además, más procesos no significan mejor rendimiento; abrir demasiados procesos aumentará el costo del cambio de procesos y tendrá un cierto impacto en el rendimiento.
