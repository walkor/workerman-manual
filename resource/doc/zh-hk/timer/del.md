```php
boolean \Workerman\Timer::del(int $timer_id)
```
刪除某個定時器

### 參數
 ``` timer_id ```

定時器的id，即add接口返回的整型

### 返回值
boolean


### 示例
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 開啟多少個進程運行定時任務，注意多進程並發問題
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // 每2秒運行一次
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // 20秒後運行一個一次性任務，刪除2秒一次的定時任務
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// 運行worker
Worker::runAll();
```

### 實例(定時器回調中刪除當前定時器)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 注意，回調裡面使用當前定時器id必須使用引用(&)的方式引入
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // 運行10次後刪除定時器
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// 運行worker
Worker::runAll();
```
