# Phương thức reConnect

```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Yêu cầu Workerman phiên bản >= 3.3.5) ```

Kết nối lại. Thường được gọi trong callback ```onClose```, để thực hiện việc kết nối lại khi mất kết nối.

Nếu mất kết nối do vấn đề mạng hoặc dịch vụ đối phương khởi động lại, bạn có thể gọi phương thức này để thực hiện việc kết nối lại.

### Tham số
``` $delay ```

Độ trễ trước khi thực hiện việc kết nối lại. Đơn vị là giây, hỗ trợ số thập phân, có thể chính xác đến mili giây.

Nếu không truyền hoặc giá trị là 0 thì đại diện cho việc kết nối lại ngay lập tức.

Nên truyền tham số để trì hoãn việc kết nối lại, tránh gây ra tình trạng độ CPU máy chủ quá cao do không thể kết nối với dịch vụ đối phương.

### Giá trị trả về
Không có giá trị trả về

### Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Nếu kết nối bị đóng, thực hiện kết nối lại sau 1 giây
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Chú ý**
> Sau khi gọi reconnect thành công, phương thức onConnect của $con sẽ được gọi lại (nếu đã được cài đặt). Đôi khi chúng ta muốn phương thức onConnect chỉ được thực hiện một lần, không thực hiện lại khi kết nối lại, tham khảo ví dụ dưới đây:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Nếu kết nối bị đóng, thực hiện kết nối lại sau 1 giây
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
