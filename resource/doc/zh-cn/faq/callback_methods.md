# PHP几种回调写法
PHP里通过匿名函数写回调是最方便的，但是除了匿名函数方式的回调，PHP还有其它的回调写法。以下是PHP几种回调写法的示例。

## 1、匿名函数回调
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 匿名函数回调
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
};

Worker::runAll();
```

## 2、普通函数回调
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 匿名函数回调
$http_worker->onMessage = 'on_message';

// 普通函数
function on_message(TcpConnection $connection, Request $request)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
}

Worker::runAll();
```

## 3、类方法作为回调
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
启动脚本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 载入MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 创建一个对象
$my_object = new MyClass();

// 调用类的方法
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

注意：
以上的代码结构不允许在构造函数里初始化资源(MySQL连接、Redis连接、Memcache连接等)，因为```$my_object = new MyClass();```运行在主进程。以MySQL为例，在主进程初始化的MySQL连接等资源会被子进程继承，每个子进程都可以操作这个数据库连接，但是这些连接在MySQL服务端对应的是同一个连接，会发生不可预期的错误，例如```mysql gone away``` 错误。

以上代码结构如果需要在类的构造函数里初始化资源，可以采用以下写法。
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // 假设数据库连接类是MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
启动脚本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 在onWorkerStart里初始化类
$worker->onWorkerStart = function($worker) {
    // 载入MyClass
    require_once __DIR__.'/MyClass.php';
    
    // 创建一个对象
    $my_object = new MyClass();

    // 调用类的方法
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

上面代码结构中onWorkerStart运行时已经是属于子进程，等于每个子进程各自建立自己的MySQL连接，所以不会有共享连接的问题。
这样还有一个好处就是支持业务代码reload。由于MyClass.php是在子进程载入的，根据reload规则业务更改MyClass.php后直接reload即可生效。

## 4、类的静态方法作为回调
静态类MyClass.php
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
启动脚本 start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 载入MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// 调用类的静态方法。
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// 如果类带命名空间，则是类似这样的写法
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

注意：根据PHP的运行机制，如果没用调用new 则不会调用构造函数，另外静态类的方法里面不允许使用```$this```。


