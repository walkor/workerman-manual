# Phương thức __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Tạo một đối tượng kết nối không đồng bộ.

AsyncTcpConnection cho phép Workerman hoạt động như một client gửi yêu cầu kết nối không đồng bộ đến máy chủ từ xa, và gửi và xử lý dữ liệu kết nối không đồng bộ thông qua giao diện gửi và gọi lại onMessage.

## Tham số
Tham số: `remote_address`

Địa chỉ kết nối, ví dụ
 ```
 tcp://www.baidu.com:80
 ssl://www.baidu.com:443
 ws://echo.websocket.org:80
 frame://192.168.1.1:8080
 text://192.168.1.1:8080
 ```

Tham số: `$context_option`

```
Tham số yêu cầu (workerman >= 3.3.5)
```

Dùng để cài đặt ngữ cảnh socket, ví dụ sử dụng `bindto` để thiết lập IP và cổng (card mạng) nào để truy cập mạng bên ngoài, thiết lập chứng chỉ SSL, v.v.

Tham khảo [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Tùy chọn ngữ cảnh socket](https://php.net/manual/zh/context.socket.php),[Tùy chọn ngữ cảnh SSL](https://php.net/manual/zh/context.ssl.php)

## Lưu ý

Hiện tại, AsyncTcpConnection hỗ trợ các giao thức [tcp](https://baike.baidu.com/subview/32754/8048820.htm),[ssl](https://baike.baidu.com/view/525499.htm),[ws](appendices/about-ws.md),[frame](appendices/about-frame.md),[text](appendices/about-text.md).

Ngoài ra còn hỗ trợ tùy chỉnh giao thức, xem [Làm thế nào để tùy chỉnh giao thức](../protocols/how-protocols.md)

Trong đó, [ssl](https://baike.baidu.com/view/525499.htm) yêu cầu Workerman>=3.3.4, và cài đặt [openssl mở rộng](https://php.net/manual/zh/book.openssl.php).

Hiện tại không hỗ trợ AsyncTcpConnection của giao thức [http](https://baike.baidu.com/view/9472.htm).

Có thể sử dụng `new AsyncTcpConnection('ws://...')` giống như trình duyệt để tạo kết nối websocket từ xa đến máy chủ websocket từ xa trong workerman, xem [Ví dụ](../appendices/about-ws.md). Tuy nhiên, không thể sử dụng `new AsyncTcpConnection('websocket://...')` để tạo kết nối websocket trong workerman.

## Ví dụ

### Ví dụ 1: Gửi yêu cầu kết nối không đồng bộ đến dịch vụ http bên ngoài
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();

$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Kết nối thành công\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Kết nối đóng\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Mã lỗi: $code Thông báo: $msg\n";
    };
    $connection_to_baidu->connect();
};

Worker::runAll();
```

### Ví dụ 2: Gửi yêu cầu kết nối không đồng bộ đến dịch vụ websocket bên ngoài và thiết lập IP và cổng để truy cập
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    $context_option = array(
        'socket' => array(
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Ví dụ 3: Gửi yêu cầu kết nối không đồng bộ đến cổng wss bên ngoài và thiết lập chứng chỉ SSL cục bộ
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    $context_option = array(
        'socket' => array(
            'bindto' => '114.215.84.87:2333',
        ),
        'ssl' => array(
            'local_cert'        => '/your/path/to/pemfile',
            'passphrase'        => 'your_pem_passphrase',
            'allow_self_signed' => true,
            'verify_peer'       => false
        )
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
