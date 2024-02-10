# Là một client ws/wss

Đôi khi cần để workerman hoạt động như một client với giao thức ws/wss để kết nối với một máy chủ và tương tác với nó.
Dưới đây là một ví dụ.

## Workerman hoạt động như client ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Sau khi bắt tay WebSocket thành công
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // Khi nhận được tin nhắn
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman hoạt động như client wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl cần truy cập vào cổng 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Thiết lập truy cập bằng cách mã hóa ssl để trở thành wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


## Workerman hoạt động như client wss (ws+ssl) cùng với chứng chỉ ssl từ máy chủ cục bộ

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Thiết lập địa chỉ IP và cổng cục bộ của máy chủ kia cùng với chứng chỉ ssl
    $context_option = array(
        // Lựa chọn ssl, xem chi tiết tại  http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Đường dẫn đến chứng chỉ cục bộ. Phải là định dạng PEM và bao gồm chứng chỉ cục bộ và khóa riêng tư.
            'local_cert'        => '/đường/dẫn/của/tập/tin/pemfile',
            // Mật khẩu của tập tin local_cert.
            'passphrase'        => 'mật_khẩu_của_pem',
            // Cho phép chứng chỉ tự ký.
            'allow_self_signed' => true,
            // Có cần xác minh chứng chỉ SSL.
            'verify_peer'       => false
        )
    );

    // ssl cần truy cập vào cổng 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Thiết lập truy cập bằng cách mã hóa ssl để trở thành wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


## Các thiết lập khác
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Khi quá trình hoạt động bắt đầu
$worker->onWorkerStart = function()
{
    // Kết nối với máy chủ websocket từ xa sử dụng giao thức websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Gửi opcode là 0x9 cho websocket hàng 55 giây một lần
    $ws_connection->websocketPingInterval = 55;
    // Tùy chỉnh tiêu đề HTTP
    $ws_connection->headers = ['token' => 'value'];
    // Thiết lập kiểu dữ liệu, mặc định là BINARY_TYPE_BLOB (văn bản)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB là văn bản, BINARY_TYPE_ARRAYBUFFER là nhị phân
    // Khi TCP hoàn tất bắt tay 3 lần
    $ws_connection->onConnect = function($connection){
        echo "đã kết nối TCP\n";
    };
    // Khi hoàn tất bắt tay của websocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Khi nhận được tin nhắn từ máy chủ websocket từ xa
    $ws_connection->onMessage = function($connection, $data){
        echo "nhận: $data\n";
    };
    // Khi xảy ra lỗi kết nối, thường là lỗi kết nối với máy chủ websocket từ xa
    $ws_connection->onError = function($connection, $code, $msg){
        echo "lỗi: $msg\n";
    };
    // Khi kết nối với máy chủ websocket từ xa bị đóng
    $ws_connection->onClose = function($connection){
        echo "kết nối đã đóng và thử kết nối lại\n";
        // Nếu kết nối bị đóng, thử kết nối lại sau 1 giây
        $connection->reConnect(1);
    };
    // Sau khi thiết lập các cấu hình trên, tiến hành thực hiện kết nối
    $ws_connection->connect();
};
Worker::runAll();
```
