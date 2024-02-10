```php
int \Workerman\Timer::sleep(float $delay)
```
Similar to the built-in `sleep()` function in PHP, the difference is that `Timer::sleep()` is non-blocking (it will not block the current process).

> **Note**
> This feature requires workerman >= 5.0
> This feature requires installation of `composer require revolt/event-loop ^1.0.0`, or use Swoole/Swow as the event-driven driver.


### Parâmetros
 ``` delay ```

Intervalo de tempo para execução, em segundos, suporta números decimais, pode ser preciso até 0.001, ou seja, preciso até a milissegundo.

### Valor de retorno
Sem valor de retorno

### Exemplo

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
