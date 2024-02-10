# PHP幾種回調寫法
在PHP中，匿名函數是最方便的回調寫法，但除了匿名函數方式的回調，PHP還有其他的回調寫法。以下是PHP幾種回調寫法的示例。

## 1、匿名函數回調
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 匿名函數回調
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // 向瀏覽器發送hello world
    $connection->send('hello world');
};

Worker::runAll();
```

## 2、普通函數回調
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 匿名函數回調
$http_worker->onMessage = 'on_message';

// 普通函數
function on_message(TcpConnection $connection, Request $request)
{
    // 向瀏覽器發送hello world
    $connection->send('hello world');
}

Worker::runAll();
```

## 3、類方法作為回調
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
啟動腳本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 載入MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 創建一個對象
$my_object = new MyClass();

// 調用類的方法
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

注意：
以上的代碼結構不允許在構造函數裡初始化資源（例如MySQL連接、Redis連接、Memcache連接等），因為```$my_object = new MyClass();```運行在主進程。以MySQL為例，在主進程初始化的MySQL連接等資源會被子進程繼承，每個子進程都可以操作這個資料庫連接，但這些連接在MySQL服務端對應的是同一個連接，會發生不可預期的錯誤，例如```mysql gone away```錯誤。

如果以上的代碼結構需要在類的構造函數裡初始化資源，可以採用以下寫法。
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // 假設資料庫連接類是MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
啟動腳本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 在onWorkerStart裡初始化類
$worker->onWorkerStart = function($worker) {
    // 載入MyClass
    require_once __DIR__.'/MyClass.php';
    
    // 創建一個對象
    $my_object = new MyClass();

    // 調用類的方法
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

上面的代碼結構中onWorkerStart運行時已經是屬於子進程，等於每個子進程各自建立自己的MySQL連接，所以不會有共享連接的問題。這樣還有一個好處就是支持業務代碼reload。由於MyClass.php是在子進程載入的，根據reload規則業務更改MyClass.php後直接reload即可生效。

## 4、類的靜態方法作為回調
靜態類MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
啟動腳本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 載入MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 調用類的靜態方法。
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// 如果類帶命名空間，則是類似這樣的寫法
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

注意：根據PHP的運行機制，如果沒有調用new則不會調用構造函數，另外靜態類的方法裡面不允許使用```$this```。
