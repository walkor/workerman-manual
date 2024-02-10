# onMessage
## Giải thích:

```php
callback Connection::$onMessage
```


Chức năng tương tự như callback [Worker::$onMessage](../worker/on-message.md), khác biệt là chỉ có hiệu lực đối với kết nối hiện tại, nghĩa là có thể thiết lập callback onMessage cho một kết nối cụ thể.


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Khi có sự kiện kết nối từ client
$worker->onConnect = function(TcpConnection $connection)
{
    // Thiết lập callback onMessage cho kết nối
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('nhận được thành công');
    };
};
// Chạy worker
Worker::runAll();
```

Mã trên tương đương với mã dưới đây

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Thiết lập callback onMessage cho tất cả kết nối trực tiếp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('nhận được thành công');
};
// Chạy worker
Worker::runAll();
```
