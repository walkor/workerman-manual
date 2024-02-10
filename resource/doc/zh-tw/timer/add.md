```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
定期執行某個函數或類方法。

注意：計時器是在當前進程中運行的，Workerman中不會創建新的進程或者線程去運行計時器。

### 參數
 ``` time_interval ```

多長時間執行一次，單位秒，支持小數，可以精確到0.001，即精確到毫秒級別。


 ``` callback ```

回調函數```注意：如果回調函數是類的方法，則方法必須是public屬性``` 

 ``` args ```

回調函數的參數，必須為數組，數組元素為參數值

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


### 2、只在指定進程中設置定時器

一個worker實例有4個進程，只在id編號為0的進程上設置定時器。

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    // 只在id編號為0的進程上設置定時器，其它1、2、3號進程不設置定時器
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4個worker進程，只在0號進程設置定時器\n";
        });
    }
};
// 運行worker
Worker::runAll();
```


### 3、定時函數為匿名函數，利用閉包傳遞參數
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// 連接建立時給對應連接設置定時器
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // 每10秒執行一次
    $time_interval = 10;
    $connect_time = time();
    // 給connection對象臨時添加一個timer_id屬性保存定時器id
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// 連接關閉時，刪除對應連接的定時器
$ws_worker->onClose = function(TcpConnection $connection)
{
    // 刪除定時器
    Timer::del($connection->timer_id);
};

// 運行worker
Worker::runAll();
```


### 4、定時函數為匿名函數，利用定時器接口傳遞參數
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// 連接建立時給對應連接設置定時器
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // 每10秒執行一次
    $time_interval = 10;
    $connect_time = time();
    // 給connection對象臨時添加一個timer_id屬性保存定時器id
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};
// 連接關閉時，刪除對應連接的定時器
$ws_worker->onClose = function(TcpConnection $connection)
{
    // 刪除定時器
    Timer::del($connection->timer_id);
};

// 運行worker
Worker::runAll();
```


### 5、定時函數為普通函數
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

// 普通的函數
function send_mail($to, $content)
{
    echo "send mail ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 10秒後執行發送郵件任務，最後一個參數傳遞false，表示只運行一次
    Timer::add(10, 'send_mail', array($to, $content), false);
};

// 運行worker
Worker::runAll();
```


### 6、定時函數為類的方法
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意，回調函數屬性必須是public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒後發送一次郵件
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

// 運行worker
Worker::runAll();
```


### 7、定時函數為類方法（類內部使用定時器）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意，回調函數屬性必須是public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }

    public function sendLater($to, $content)
    {
        // 回調的方法屬於當前的類，則回調數組第一個元素為$this
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後發送一次郵件
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    $mail->sendLater($to, $content);
};

// 運行worker
Worker::runAll();
```


### 8、定時函數為類的靜態方法
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意這個是靜態方法，回調函數屬性也必須是public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後發送一次郵件
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 定時調用類的靜態方法
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

// 運行worker
Worker::runAll();
```


### 9、定時函數為類的靜態方法(帶命名空間)
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意這個是靜態方法，回調函數屬性也必須是public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後發送一次郵件
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 定時調用帶命名空間的類的靜態方法
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

// 運行worker
Worker::runAll();
```


### 10、定時器中銷毀當前定時器（use閉包方式傳遞$timer_id）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 計數
    $count = 1;
    // 要想$timer_id能正確傳遞到回調函數內部，$timer_id前面必須加地址符 &
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Timer run $count\n";
        // 運行10次後銷毀當前定時器
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

// 運行worker
Worker::runAll();
```


### 11、定時器中銷毀當前定時器（參數方式傳遞$timer_id）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content, $timer_id)
    {
        // 臨時給當前對象添加一個count屬性，記錄定時器運行次數
        $this->count = empty($this->count) ? 1 : $this->count;
        // 運行10次後銷毀當前定時器
        echo "send mail {$this->count}...\n";
        if($this->count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new Mail();
    // 要想$timer_id能正確傳遞到回調函數內部，$timer_id前面必須加地址符 &
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

// 運行worker
Worker::runAll();
```

