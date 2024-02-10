# Cách gửi và nhận dữ liệu hex trong webman

**Nhận dữ liệu hex**

Khi nhận dữ liệu, bạn có thể sử dụng hàm `bin2hex($data)` để chuyển đổi dữ liệu sang định dạng hexadeciaml (16 cơ số).

**Gửi dữ liệu hex**

Trước khi gửi dữ liệu, hãy sử dụng `hex2bin($data)` để chuyển đổi dữ liệu hex sang dạng nhị phân trước khi gửi.

**Ví dụ:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Chuyển dữ liệu sang định dạng hex
    $hex_data = bin2hex($data);
    // Gửi dữ liệu hex đến client
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
