# add
## Description:
```php
int \Workerman\Lib\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
To schedule execution of a callback after/every ```$time_interval``` seconds. Returns a timerId for possible use with Timer::del(). Optionally you can also pass arguments to the callback.

## Notice
You should add Timer at ```on{.....}```


### Parameters
``` time_interval ```

The delay time. For example 2.5, 1, 0.01


``` callback ```

Callback

``` args ```

Arguments

``` persistent ```

If the execution is one-time callback, ``` persistent ``` should be set ```false```, otherwise ``` persistent ``` should be set ```true```.

### Return Values
Return a integer as timerid

### Examples
```php
use \Workerman\Worker;

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $time_interval = 2.5;
    \Workerman\Lib\Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

```
