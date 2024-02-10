# Các cách viết callback trong PHP
Trong PHP, việc sử dụng hàm ẩn danh để viết callback là cách tiện lợi nhất, tuy nhiên ngoài cách viết callback bằng hàm ẩn danh, PHP còn có một số cách viết callback khác. Dưới đây là một số ví dụ về cách viết callback trong PHP.

## 1. Callback bằng hàm ẩn danh
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback bằng hàm ẩn danh
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Gửi chuỗi 'hello world' đến trình duyệt
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Callback bằng hàm thông thường
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback bằng hàm thông thường
$http_worker->onMessage = 'on_message';

// Hàm thông thường
function on_message(TcpConnection $connection, Request $request)
{
    // Gửi chuỗi 'hello world' đến trình duyệt
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Sử dụng phương thức của lớp làm callback
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
Script khởi động start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Tải lớp MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Tạo một đối tượng
$my_object = new MyClass();

// Gọi phương thức của lớp
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Lưu ý: Cấu trúc mã trên không cho phép khởi tạo tài nguyên (kết nối MySQL, kết nối Redis, kết nối Memcache, v.v.) trong hàm tạo vì ```$my_object = new MyClass();``` chạy trong quá trình chính. Ví dụ với kết nối MySQL, tài nguyên khởi tạo trong quá trình chính sẽ được thừa kế bởi tiến trình con và mỗi tiến trình con có thể điều khiển kết nối cơ sở dữ liệu này. Tuy nhiên, các kết nối này tương ứng với một kết nối duy nhất trên máy chủ MySQL, dẫn đến lỗi không thể dự đoán như "mysql gone away".

Nếu cấu trúc mã trên cần khởi tạo tài nguyên trong hàm tạo của lớp, có thể sử dụng cách viết sau.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Giả sử lớp kết nối cơ sở dữ liệu là MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Script khởi động start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Khởi tạo lớp trong onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Tải lớp MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Tạo một đối tượng
    $my_object = new MyClass();

    // Gọi phương thức của lớp
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

Trong cấu trúc mã trên, khi onWorkerStart chạy, đó đã thuộc về tiến trình con, mỗi tiến trình con tự xây dựng kết nối MySQL riêng, do đó không có vấn đề kết nối chung. Một lợi ích khác là hỗ trợ reload mã nguồn. Vì MyClass.php được tải trong tiến trình con, sau khi thay đổi mã nguồn, chỉ cần reload là có hiệu lực.

## 4. Sử dụng phương thức tĩnh của lớp làm callback
MyClass.php tĩnh
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
Script khởi động start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Tải lớp MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Gọi phương thức tĩnh của lớp.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Nếu lớp có namespace, cách sử dụng sẽ tương tự là
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

Lưu ý: Theo cách PHP hoạt động, nếu không gọi new thì hàm tạo không được gọi, và các phương thức của lớp tĩnh không được phép sử dụng ```$this```.
