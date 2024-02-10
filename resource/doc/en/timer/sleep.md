# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```

Similar to PHP's built-in `sleep()` function, the difference is that `Timer::sleep()` is non-blocking (it does not block the current process).

> **Note**
> This feature requires workerman>=5.0
> This feature requires installation of composer require revolt/event-loop ^1.0.0, or use Swoole/Swow as an event driver


### Parameters
 ``` delay ```

How long to execute, in seconds, supports decimals, down to 0.001, i.e., precise to the millisecond level.

### Return
No return value

### Example

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
