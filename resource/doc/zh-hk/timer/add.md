```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
定期執行某個函數或者類方法。

注意：定時器是在當前進程中運行的，workerman中不會創建新的進程或者線程去運行定時器。

### 參數
 ``` time_interval ```

多長時間執行一次，單位秒，支持小數，可以精確到0.001，即精確到毫秒級別。


 ``` callback ```

回調函數```注意：如果回調函數是類的方法，則方法必須是public屬性```


 ``` args ```

回調函數的參數，必須為陣列，陣列元素為參數值


 ``` persistent ```

是否是持久的，如果只想定時執行一次，則傳遞false（只執行一次的任務在執行完畢後會自動銷毀，不必調用```Timer::del()```）。默認是true，即一直定時執行。

### 返回值
返回一個整數，代表計時器的timerid，可以通過調用```Timer::del($timerid)```銷毀這個計時器。

### 示例

#### 1、定時函數為匿名函數（閉包）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 開啟多少個進程運行定時任務，注意業務是否在多進程有並發問題
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // 每2.5秒執行一次
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// 運行worker
Worker::runAll();
```
