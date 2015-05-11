# add
```php
int \Workerman\Lib\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
定时执行某个函数或者类方法

### 参数
``` time_interval ```

多长时间执行一次，单位秒，支持小数，可以精确到0.001，即精确到毫秒级别。


``` callback ```

回调函数

``` args ```

回调函数的参数，必须为数组

``` persistent ```

是否是持久的，如果只想定时执行一次，则传递false。默认是true，即一直定时执行。

### 返回值
返回一个整数，代表计时器的timerid，可以通过这个timerid删除对应的计时器

### 示例
```php
use \Workerman\Worker;

$task = new Worker();
// 开启多少个进程运行定时任务，注意多进程并发问题
$task->count = 1;
$task->onWorkerStart = function($task)
{
    $time_interval = 2.5;
    \Workerman\Lib\Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// 运行worker
Worker::runAll();
```
