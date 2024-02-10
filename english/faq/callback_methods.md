# Several Callback Writing Methods in PHP
In PHP, using anonymous functions to write callbacks is the most convenient way, but in addition to using anonymous functions, there are other ways to write callbacks in PHP. Below are examples of several callback writing methods in PHP.

## 1. Anonymous Function Callback

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Anonymous function callback
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Send 'hello world' to the browser
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Regular Function Callback

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Anonymous function callback
$http_worker->onMessage = 'on_message';

// Regular function
function on_message(TcpConnection $connection, Request $request)
{
    // Send 'hello world' to the browser
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Using Class Methods as Callbacks

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
Start script start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Include MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Create an object
$my_object = new MyClass();

// Call the class methods
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Please note that the above code structure does not allow resource initialization (such as MySQL connection, Redis connection, Memcache connection, etc.) in the constructor, because ```$my_object = new MyClass();``` runs in the main process. For example, in the case of MySQL, resources initialized in the main process, such as database connections, will be inherited by the child processes, and each child process can operate on the same database connection. However, these connections correspond to the same connection on the MySQL server, which can lead to unexpected errors, such as the "mysql gone away" error.

If resource initialization in the constructor of the class is needed, the following writing method can be used.

MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

class MyClass{
    protected $db = null;
    public function __construct(){
        // Assuming the database connection class is MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Start script start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Initialize the class in onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Include MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Create an object
    $my_object = new MyClass();

    // Call the class methods
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

In the code structure above, when onWorkerStart is executed, it already belongs to the child process, which means that each child process establishes its own MySQL connection, so there are no issues with shared connections. Another benefit of this approach is support for business code reload. Since MyClass.php is loaded in the child process, any modifications to the business code in MyClass.php will take effect directly after a reload according to the reload rules.

## 4. Using Class Static Methods as Callbacks

Static class MyClass.php
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
Start script start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Include MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Call the class static methods
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// If the class has a namespace, the usage would be something like this
// $worker->onWorkerStart = array('your\namespace\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namespace\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namespace\MyClass', 'onMessage');
// $worker->onClose       = array('your\namespace\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namespace\MyClass', 'onWorkerStop');

Worker::runAll();
```

Please note that according to the PHP execution mechanism, if the constructor is not called, it means that the constructor function will not be called, and static class methods do not allow the use of ```$this```.
