# del
```php
boolean \Workerman\Lib\Timer::del(int $timer_id)
```
删除某个定时器

### 参数
``` timer_id ```

定时器的id，即add接口返回的整型

### 返回值
boolean


### 示例
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

$task = new Worker();
// 开启多少个进程运行定时任务，注意多进程并发问题
$task->count = 1;
$task->onWorkerStart = function($task)
{
    // 每2秒运行一次
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // 20秒后运行一个一次性任务，删除2秒一次的定时任务
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// 运行worker
Worker::runAll();
```

### 实例(定时器回调中删除当前定时器)
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 注意，回调里面使用当前定时器id必须使用引用(&)的方式引入
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // 运行10次后删除定时器
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// 运行worker
Worker::runAll();
```
