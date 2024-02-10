# defaultMaxSendBufferSize
## Giải thích:
```php
static int Connection::$defaultMaxSendBufferSize
```

Thuộc tính này là một thuộc tính tĩnh toàn cục, được sử dụng để thiết lập kích thước bộ đệm gửi ứng dụng mặc định của tất cả kết nối. Nếu không được thiết lập, giá trị mặc định là ```1MB```. ```Connection::$defaultMaxSendBufferSize``` có thể được thiết lập động, và sau khi được thiết lập, nó chỉ có hiệu lực đối với các kết nối mới phát sinh sau đó.

Thuộc tính này ảnh hưởng đến callback [onBufferFull](../worker/on-buffer-full.md).

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Thiết lập kích thước bộ đệm gửi ứng dụng mặc định cho tất cả kết nối
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Thiết lập kích thước bộ đệm gửi ứng dụng của kết nối hiện tại, sẽ ghi đè lên giá trị mặc định
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Chạy worker
Worker::runAll();
```
