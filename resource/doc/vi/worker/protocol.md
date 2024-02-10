# giao thức
Yêu cầu ```（workerman >= 3.2.7）```

## Mô tả:
```php
string Worker::$protocol
```

Thiết lập lớp giao thức cho phiên bản Worker hiện tại.

Chú ý: Lớp xử lý giao thức có thể được chỉ định trực tiếp khi khởi tạo Worker trong tham số lắng nghe. Ví dụ
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Đoạn code trên tương đương với đoạn code dưới đây

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Đầu tiên, nó sẽ kiểm tra xem người dùng có lớp giao thức tùy chỉnh \Protocols\Http hay không,
 * Nếu không, sẽ sử dụng lớp giao thức tích hợp trong workerman Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```
