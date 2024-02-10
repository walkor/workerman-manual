# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
Similar to the PHP built-in `sleep()` function, the difference is that `Timer::sleep()` is non-blocking (it won't block the current process).

> **Note**
> This feature requires workerman >= 5.0.
> This feature requires installation of `composer require revolt/event-loop ^1.0.0`, or use Swoole/Swow as the event driver.

### Parámetros
``` delay ```

Indica el tiempo en segundos para ejecutar la función, admite números decimales y puede ser preciso hasta 0.001, es decir, a nivel de milisegundos.

### Valor de retorno
No tiene valor de retorno.

### Ejemplo

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Delay sending data by 1.5 seconds
    Timer::sleep(1.5);
    // Send data
    $connection->send('hello workerman');
};

Worker::runAll();
```
