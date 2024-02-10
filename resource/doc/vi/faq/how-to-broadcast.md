# Cách phát sóng (broadcast) dữ liệu

## Ví dụ (Phát sóng theo thời gian)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Trong ví dụ này, số tiến trình phải là 1
$worker->count = 1;
// Khi tiến trình khởi động, thiết lập một bộ định thời, gửi dữ liệu đến tất cả các kết nối khách hàng định kỳ
$worker->onWorkerStart = function($worker)
{
    // Định kỳ, mỗi 10 giây một lần
    Timer::add(10, function() use ($worker)
    {
        // Duyệt qua tất cả các kết nối khách hàng của tiến trình hiện tại, gửi thời gian hiện tại của máy chủ
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Chạy worker
Worker::runAll();
```

## Ví dụ (Trò chuyện nhóm)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Trong ví dụ này, số tiến trình phải là 1
$worker->count = 1;
// Khi khách hàng gửi tin nhắn, phát sóng cho các khách hàng khác
$worker->onMessage = function(TcpConnection $connection, $message) use ($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Chạy worker
Worker::runAll();
```

## Giải thích:
**Một tiến trình:** Các ví dụ trên chỉ có thể hoạt động với **một tiến trình** (```$worker->count=1```), vì khi có nhiều tiến trình, nhiều khách hàng có thể kết nối đến các tiến trình khác nhau, và các khách hàng giữa các tiến trình được tách rời và không thể trực tiếp giao tiếp, có nghĩa là tiến trình A không thể **trực tiếp** điều khiển đối tượng kết nối của khách hàng trong tiến trình B để gửi dữ liệu. (Để làm được điều này, cần sử dụng giao tiếp giữa các tiến trình, ví dụ có thể sử dụng thành phần Channel, ví dụ [ví dụ-gửi dữ liệu cụm](../components/channel-examples.md), [ví dụ-gửi dữ liệu theo nhóm](../components/channel-examples2.md)).

**Nên sử dụng GatewayWorker:** GatewayWoker, được phát triển dựa trên workerman, cung cấp cơ chế đẩy dữ liệu thuận tiện hơn, bao gồm phát sóng theo nhóm, phát sóng và có thể cài đặt nhiều tiến trình hoặc triển khai nhiều máy chủ. Nếu cần đẩy dữ liệu đến khách hàng, nên sử dụng framework GatewayWorker.

GatewayWorker manual: https://doc2.workerman.net
