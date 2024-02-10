```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
``` 
指定された関数またはクラスメソッドを定期的に実行します。

注意: タイマーは現在のプロセスで実行され、workermanでは新しいプロセスやスレッドを作成しません。

### パラメーター
 ``` time_interval ``` 

実行間隔、単位は秒で小数をサポートします。0.001秒（ミリ秒）まで精度を設定できます。

 ``` callback ``` 

コールバック関数```注意：コールバック関数がクラスメソッドの場合、メソッドはpublicである必要があります。```

 ``` args ``` 

コールバック関数のパラメータは、配列でなければなりません。配列の要素はパラメータ値です。

 ``` persistent ``` 

永続的かどうか。一度だけ実行したい場合はfalseを渡します（一度だけ実行されたタスクは終了後に自動的に破棄され、```Timer::del()```を呼び出す必要がありません）。デフォルトはtrueで、常に定期的に実行されます。

### 戻り値
タイマーのtimeridを表す整数を返します。これを使用して```Timer::del($timerid)```を呼び出してタイマーを破棄できます。

### 例

#### 1、タイミング関数が匿名関数（クロージャ）の場合
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// タイミングタスクを複数のプロセスで実行するために、複数プロセスでビジネスを実行するなどに注目してください
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // 2.5秒ごとに実行
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Workerを起動
Worker::runAll();
```

### 2、指定したプロセスのみでタイマーを設定する

1つのworkerインスタンスには4つのプロセスがあり、ID番号が0のプロセスにのみタイマーを設定します。

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    // ID番号が0のプロセスにのみタイマーを設定し、1、2、3号プロセスにはタイマーを設定しない
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4つのworkerプロセス、0番プロセスのみタイマーを設定\n";
        });
    }
};
// Workerを起動
Worker::runAll();
```

### 3、タイミング関数が匿名関数で、クロージャを使用してパラメータを渡す
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// 接続が確立されたら、対応する接続にタイマーを設定します
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // 10秒ごとに実行
    $time_interval = 10;
    $connect_time = time();
    // connectionオブジェクトに一時的にtimer_idプロパティを追加して、タイマーIDを保存します
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// 接続が切断されたら、対応する接続のタイマーを削除
$ws_worker->onClose = function(TcpConnection $connection)
{
    // タイマーを削除
    Timer::del($connection->timer_id);
};

// Workerを起動
Worker::runAll();
```

### 4、タイミング関数が匿名関数、タイマーインターフェースを使用してパラメータを渡す
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// 接続が確立されたら、対応する接続にタイマーを設定
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // 10秒ごとに実行
    $time_interval = 10;
    $connect_time = time();
    // connectionオブジェクトに一時的にtimer_idプロパティを追加して、タイマーIDを保存します
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};
// 接続が切断されたら、対応する接続のタイマーを削除
$ws_worker->onClose = function(TcpConnection $connection)
{
    // タイマーを削除
    Timer::del($connection->timer_id);
};

// Workerを起動
Worker::runAll();
```

### 5、タイミング関数が通常の関数の場合
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

// 通常の関数
function send_mail($to, $content)
{
    echo "send mail ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 10秒後にメールを送信するタスク。最後のパラメーターをfalseに渡すと、1回だけ実行されます
    Timer::add(10, 'send_mail', array($to, $content), false);
};

// Workerを起動
Worker::runAll();
```

### 6、タイミング関数がクラスメソッドの場合
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意: コールバック関数属性はpublicである必要があります
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 10秒後にメールを送信する
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

// Workerを起動
Worker::runAll();
```

### 7、タイミング関数がクラスメソッド（クラス内でタイマー使用）の場合
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // 注意: コールバック関数属性はpublicである必要があります
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }

    public function sendLater($to, $content)
    {
        // コールバックのメソッドは現在のクラスに属している場合、コールバック配列の最初の要素は$thisであることに注意
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後にメールを送信する
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    $mail->sendLater($to, $content);
};

// Workerを起動
Worker::runAll();
```

### 8、タイミング関数がクラスの静的メソッドの場合
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // これは静的メソッドです。コールバック関数属性もpublicである必要があります
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後にメールを送信する
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // クラスの静的メソッドを定期的に呼び出す
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

// Workerを起動
Worker::runAll();
```

### 9、タイミング関数がクラスの静的メソッド（名前空間を含む）の場合
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // これは静的メソッドです。コールバック関数属性もpublicである必要があります
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 10秒後にメールを送信する
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // 名前空間を含むクラスの静的メソッドを定期的に呼び出す
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

// Workerを起動
Worker::runAll();
```

### 10、タイミング関数内で現在のタイマーを破棄（useクロージャで$timer_idを渡す方法）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // カウント
    $count = 1;
    // $timer_idがコールバック関数内部に正しく渡されるようにするため、$timer_idの前にアドレス記号 & を付ける必要があります
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Timer run $count\n";
        // 10回実行した後に現在のタイマーを破棄
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

// Workerを起動
Worker::runAll();
```

### 11、タイミング関数内で現在のタイマーを破棄（パラメータで$timer_idを渡す方法）
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content, $timer_id)
    {
        // 一時的に現在のオブジェクトにcount属性を追加し、タイマーの実行回数を記録する
        $this->count = empty($this->count) ? 1 : $this->count;
        // 10回実行した後に現在のタイマーを破棄
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
    // $timer_idがコールバック関数内部に正しく渡されるようにするため、$timer_idの前にアドレス記号 & を付ける必要があります
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

// Workerを起動
Worker::runAll();
```
