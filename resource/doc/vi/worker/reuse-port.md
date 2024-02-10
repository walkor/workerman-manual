# reusePort
> **Chú ý**
> Yêu cầu workerman>= 3.2.1  PHP>=7.0，Không hỗ trợ tính năng này trên hệ điều hành Windows và Mac OS.

## Miêu tả:

```php
bool Worker::$reusePort
```

Thiết lập xem worker hiện tại có mở cổng lắng nghe tái sử dụng không (tùy chọn SO_REUSEPORT của socket).

Khi mở cổng lắng nghe tái sử dụng, nhiều tiến trình không có quan hệ họ hàng có thể lắng nghe cùng một cổng và hệ thống nhân bản quyết định sẽ giao kết nối socket cho tiến trình nào xử lý, tránh hiệu ứng bầy đàn và có thể nâng cao hiệu suất các ứng dụng đa tiến trình kết nối ngắn. 

**Chú ý:** Tính năng này yêu cầu phiên bản PHP>=7.0.

## Ví dụ 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Chạy worker
Worker::runAll();
```

## Ví dụ 2: Lắng nghe nhiều cổng (nhiều giao thức) trên Workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Mỗi tiến trình sau khi khởi động sẽ thêm một lắng nghe trong tiến trình hiện tại
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Nhiều tiến trình lắng nghe cùng một cổng (socket lắng nghe không được thừa kế từ tiến trình cha)
     * Cần phải bật tái sử dụng cổng, nếu không sẽ báo lỗi "Address already in use"
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Tiến hành lắng nghe
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Chạy worker
Worker::runAll();
```
