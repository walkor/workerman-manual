# globalEvent

## Mô tả:
```php
static Event Worker::$globalEvent
```

Thuộc tính này là một thuộc tính tĩnh toàn cầu, là một phiên bản eventloop toàn cầu, có thể đăng ký sự kiện đọc và viết từ file descriptor hoặc sự kiện tín hiệu.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // Khi tiến trình nhận được tín hiệu SIGALRM, in ra một số thông tin
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Nhận được tín hiệu SIGALRM\n";
    });
};
// Chạy worker
Worker::runAll();
```

## Kiểm tra
Sau khi Workerman được khởi động, nó sẽ in ra pid hiện tại của tiến trình (một con số). Chạy lệnh sau trong dòng lệnh
```shell
kill -SIGALRM pid của tiến trình
```
Máy chủ sẽ in ra
```shell
Nhận được tín hiệu SIGALRM
```
