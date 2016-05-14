# add
```php
int \Workerman\Lib\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
定时执行某个函数或者类方法。

注意：定时器是在当前进程中运行的，workerman中不会创建新的进程或者线程去运行定时器。

### 参数
``` time_interval ```

多长时间执行一次，单位秒，支持小数，可以精确到0.001，即精确到毫秒级别。


``` callback ```

回调函数```注意：如果回调函数是类的方法，则方法必须是public属性```

``` args ```

回调函数的参数，必须为数组，数组元素为参数值

``` persistent ```

是否是持久的，如果只想定时执行一次，则传递false（只执行一次的任务在执行完毕后会自动销毁，不必调用```Timer::del()```）。默认是true，即一直定时执行。

### 返回值
返回一个整数，代表计时器的timerid，可以通过调用```Timer::del($timerid)```销毁这个计时器。

### 示例

#### 1、定时函数为匿名函数（闭包）
```php
use \Workerman\Worker;
use \Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$task = new Worker();
// 开启多少个进程运行定时任务，注意多进程并发问题
$task->count = 1;
$task->onWorkerStart = function($task)
{
    // 每2.5秒执行一次
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// 运行worker
Worker::runAll();
```

### 2、定时函数为普通函数
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

// 普通的函数
function send_mail($to, $content)
{
    echo "send mail ...\n";
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 10秒后执行发送邮件任务，最后一个参数传递false，表示只运行一次
    Timer::add(10, 'send_mail', array($to, $content), false);
};

// 运行worker
Worker::runAll();
```

### 3、定时函数为类的方法
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

class Mail
{
    // 注意，回调函数属性必须是public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒后发送一次邮件
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

// 运行worker
Worker::runAll();
```

### 4、定时函数为类方法（类内部使用定时器）
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

class Mail
{
    // 注意，回调函数属性必须是public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }

    public function sendLater($to, $content)
    {
        // 回调的方法属于当前的类，则回调数组第一个元素为$this
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒后发送一次邮件
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    $mail->sendLater($to, $content);
};

// 运行worker
Worker::runAll();
```

### 5、定时函数为类的静态方法
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

class Mail
{
    // 注意这个是静态方法，回调函数属性也必须是public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒后发送一次邮件
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 定时调用类的静态方法
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

// 运行worker
Worker::runAll();
```

### 6、定时函数为类的静态方法(带命名空间)
```php
namespace Task;
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

class Mail
{
    // 注意这个是静态方法，回调函数属性也必须是public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒后发送一次邮件
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 定时调用带命名空间的类的静态方法
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

// 运行worker
Worker::runAll();
```

### 7、定时器中销毁当前定时器（use闭包方式传递$timer_id）
```php
use \Workerman\Worker;
use \Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 计数
    $count = 1;
    // 要想$timer_id能正确传递到回调函数内部，$timer_id前面必须加地址符 &
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Timer run $count\n";
        // 运行10次后销毁当前定时器
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

// 运行worker
Worker::runAll();
```

### 8、定时器中销毁当前定时器（参数方式传递$timer_id）
```php
require_once './Workerman/Autoloader.php';
use \Workerman\Worker;
use \Workerman\Lib\Timer;

class Mail
{
    public function send($to, $content, $timer_id)
    {
        // 临时给当前对象添加一个count属性，记录定时器运行次数
        $this->count = empty($this->count) ? 1 : $this->count;
        // 运行10次后销毁当前定时器
        echo "send mail {$this->count}...\n";
        if($this->count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $mail = new Mail();
    // 要想$timer_id能正确传递到回调函数内部，$timer_id前面必须加地址符 &
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

// 运行worker
Worker::runAll();
```

### 9、只在指定进程中设置定时器

一个worker实例有4个进程，只在id编号为0的进程上设置定时器。

```php
use Workerman\Worker;
use Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // 只在id编号为0的进程上设置定时器，其它1、2、3号进程不设置定时器
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4个worker进程，只在0号进程设置定时器\n";
        });
    }
};
// 运行worker
Worker::runAll();
```
