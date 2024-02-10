# id

## Giải thích:
```php
int Connection::$id
```

Id của kết nối. Đây là một số nguyên tự tăng.

Chú ý: Workerman là multi-process, mỗi tiến trình sẽ duy trì một id kết nối tự tăng, vì vậy id kết nối giữa nhiều tiến trình sẽ có thể trùng nhau. Nếu muốn có id kết nối không trùng lặp, có thể gán lại giá trị cho connection->id theo nhu cầu, ví dụ thêm tiền tố worker->id vào đó.

## Xem thêm
[Thuộc tính connections của Worker](../worker/connections.md)

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Chạy worker
Worker::runAll();
```
