```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
تنفيذ وظيفة معينة أو طريقة كائن بشكل منتظم.

ملاحظة: يتم تشغيل خوارزمية التوقيت في نفس العملية، لا يتم إنشاء عمليات أو خيوط جديدة لتشغيل المؤقت في workerman.

### المعلمات

 ``` time_interval ```

مدة الوقت بين كل عملية، بالثواني ويُدعم الأرقام العشرية حتى الألفية (مما يعني دقة حتى الجزء الألفي من الثانية).

 ``` callback ```

وظيفة الاستدعاء ``` ملاحظة: إذا كانت وظيفة الاستدعاء طريقة للفئة، يجب أن تكون الطريقة عامة``` 

 ``` args ```

معلمات وظيفة الاستدعاء ويجب أن تكون كمصفوفة، وعناصر المصفوفة تكون قيم المعلمات.

 ``` persistent ```

هل هو دائم، إذا كنت ترغب فقط في تنفيذ الوظيفة مرة واحدة، يجب تمرير القيمة عدم الاستمرار (المهمة التي تُنفذ مرة واحدة ستتم تدميرها تلقائيًا بعد الانتهاء من التشغيل، ولا حاجة لاستدعاء ```Timer::del()```). الافتراضي هو الصحيح، أي أن التنفيذ متواصل.

### القيمة المُرجعة
تعيد الدالة قيمة صحيحة، تُمثل مُعرف المؤقت، يمكن استخدام ```Timer::del($timerid)``` لتدمير هذا المؤقت.

### مثال
#### 1. وظيفة المؤقت مثنوي (مستدعى نموذجي)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

Worker::runAll();
```

#### 2. ضبط مؤقت في عملية محددة فقط

عندما يكون لدى مثيل العامل 4 عمليات، يتم ضبط المؤقت في العملية ذات الرقم التعريفي 0 فقط.
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 عمليات عاملة، يتم ضبط المؤقت فقط في العملية ذات الرقم التعريفي 0\n";
        });
    }
};

Worker::runAll();
```

#### 3. وظيفة المؤقت مثنوية كنموذج خاص بإرسال المعلومات (عند تسجيل الاتصال)

عندما يقوم العمل ببناء الاتصال، يتم ضبط المؤقت للاتصال المقابل كل 10 ثواني.
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
$ws_worker->onConnect = function(TcpConnection $connection)
{
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};

$ws_worker->onClose = function(TcpConnection $connection)
{
    Timer::del($connection->timer_id);
};

Worker::runAll();
```

#### 4. وظيفة المؤقت كنموذج خاص لاستخدام واجهة المؤقت لتمرير المعلمة

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
$ws_worker->onConnect = function(TcpConnection $connection)
{
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};

$ws_worker->onClose = function(TcpConnection $connection)
{
    Timer::del($connection->timer_id);
};

Worker::runAll();
```

#### 5. وظيفة المؤقت كدالة عادية
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

function ارسال_بريد($الى, $المحتوى)
{
    echo "ارسال بريد ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $الى = 'workerman@workerman.net';
    $المحتوى = 'مرحباً workerman';
    Timer::add(10, 'ارسال_بريد', array($الى, $المحتوى), false);
};

Worker::runAll();
```

#### 6. وظيفة المؤقت كطريقة للصنف
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class البريد
{
    public function send($to, $content)
    {
        echo "ارسال بريد ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $mail = new البريد();
    $الى = 'workerman@workerman.net';
    $المحتوى = 'مرحباً workerman';
    Timer::add(10, array($mail, 'send'), array($الى, $المحتوى), false);
};

Worker::runAll();
```

#### 7. وظيفة المؤقت كطريقة للصنف (التواجد الداخلي لاستخدام المؤقت)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class البريد
{
    public function send($to, $content)
    {
        echo "ارسال بريد ...\n";
    }

    public function sendLater($to, $content)
    {
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new البريد();
    $الى = 'workerman@workerman.net';
    $المحتوى = 'مرحباً workerman';
    $mail->sendLater($الى, $المحتوى);
};

Worker::runAll();
```

#### 8. وظيفة المؤقت كطريقة استاتية للصنف
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class البريد
{
    public static function ارسال($الى, $المحتوى)
    {
        echo "ارسال بريد ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $الى = 'workerman@workerman.net';
    $المحتوى = 'مرحباً workerman';
    Timer::add(10, array('البريد', 'ارسال'), array($الى, $المحتوى), false);
};

Worker::runAll();
```

#### 9. وظيفة المؤقت كطريقة استاتية للصنف (باستخدام مساحة الأسماء)
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class البريد
{
    public static function ارسال($الى, $المحتوى)
    {
        echo "ارسال بريد ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $الى = 'workerman@workerman.net';
    $المحتوى = 'مرحباً workerman';
    Timer::add(10, array('\Task\البريد', 'ارسال'), array($الى, $المحتوى), false);
};

Worker::runAll();
```

#### 10. تدمير المؤقت في داخل المؤقت (استخدام use لتمرير $timer_id)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $count = 1;
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "تشغيل المؤقت $count\n";
        if($count++ >= 10)
        {
            echo "تدمير الحدّ $timer_id\n";
            Timer::del($timer_id);
        }
    });
};

Worker::runAll();
```

#### 11. تدمير المؤقت في داخل المؤقت (استخدام المعلمة لتمرير $timer_id)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class البريد
{
    public function send($to, $content, $timer_id)
    {
        $this->count = empty($this->count) ? 1 : $this->count;
        echo "ارسال بريد {$this->count}...\n";
        if($this->count++ >= 10)
        {
            echo "تدمير الحدّ $timer_id\n";
            Timer::del($timer_id);
        }
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new البريد();
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

Worker::runAll();
```
