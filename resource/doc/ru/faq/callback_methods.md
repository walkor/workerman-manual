# Несколько способов обратного вызова в PHP
В PHP наиболее удобным способом написания обратного вызова является использование анонимных функций, однако помимо анонимных функций в PHP существуют и другие способы написания обратного вызова. Ниже приведены примеры нескольких способов обратного вызова в PHP.

## 1. Обратный вызов с использованием анонимной функции
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Обратный вызов через анонимную функцию
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Отправить "hello world" на браузер
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Обратный вызов с использованием обычной функции
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Обратный вызов через обычную функцию
$http_worker->onMessage = 'on_message';

// Обычная функция
function on_message(TcpConnection $connection, Request $request)
{
    // Отправить "hello world" на браузер
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Метод класса в качестве обратного вызова
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
Start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Подключение класса MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Создание объекта класса
$my_object = new MyClass();

// Вызов методов класса
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Замечание: В приведенной структуре кода нельзя инициализировать ресурсы (соединение с MySQL, Redis, Memcache и т. д.) в конструкторе, поскольку ```$my_object = new MyClass();``` выполняется в главном процессе. Например, при инициализации соединения с MySQL в главном процессе это соединение будет унаследовано дочерними процессами, и каждый дочерний процесс сможет работать с этим соединением. Однако на стороне сервера MySQL это одно и то же соединение, что может привести к непредвиденным ошибкам, таким как ошибка ```mysql gone away```.

В данной структуре кода, если необходимо инициализировать ресурсы в конструкторе класса, можно использовать следующий способ:
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Предположим, что класс соединения с базой данных - MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Инициализация класса в onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Подключение MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Создание объекта
    $my_object = new MyClass();

    // Вызов методов класса
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

В этой структуре кода onWorkerStart уже выполняется в дочерних процессах, что означает, что каждый дочерний процесс устанавливает свое собственное соединение с MySQL, что исключает проблему общего соединения. Также это позволяет поддерживать перезагрузку кода бизнес-логики. Поскольку MyClass.php загружается в дочерних процессах, любые изменения в бизнес-логике в MyClass.php могут быть сразу же применены после перезагрузки. 

## 4. Использование статического метода класса в качестве обратного вызова
Статический класс MyClass.php
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
Стартовый скрипт start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Подключение MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Вызов статических методов класса
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Если класс имеет пространство имен, то соответствующим образом использовать
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

Замечание: В соответствии с механизмом выполнения PHP, если не был вызван new, то конструктор не будет вызван, кроме того, в статических методах класса нельзя использовать ```$this```.
