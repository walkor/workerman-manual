# vận chuyển
## Giải thích:
```php
string Worker::$transport
```

Đặt giao thức vận chuyển được sử dụng bởi Worker hiện tại, hiện tại chỉ hỗ trợ 3 loại (tcp, udp, ssl). Nếu không được thiết lập, mặc định là tcp.

``` Lưu ý: ssl yêu cầu Workerman phiên bản >= 3.3.7 ```

## Ví dụ 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Sử dụng giao thức udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Chạy worker
Worker::runAll();
```

## Ví dụ 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Chứng chỉ tốt nhất là chứng chỉ được đăng ký
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // Cũng có thể là tệp crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Ở đây đặt giao thức là websocket
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Thiết lập transport để bật ssl, websocket+ssl là wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
