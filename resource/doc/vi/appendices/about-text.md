# Giao thức văn bản

> Workerman xác định một loại giao thức văn bản gọi là "text protocol", với định dạng giao thức là ```gói dữ liệu + kí tự xuống dòng```, nghĩa là thêm kí tự xuống dòng vào cuối mỗi gói dữ liệu để biểu thị kết thúc gói.

Ví dụ với chuỗi buffer1 và buffer2 dưới đây đều phù hợp với giao thức văn bản:

```php
// Văn bản kết thúc bằng một dấu xuống dòng
$buffer1 = 'abcdefghijklmn
';
// Trong php, \n ở giữa dấu ngoặc kép đại diện cho một kí tự xuống dòng, ví dụ như "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Thiết lập kết nối socket với máy chủ
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Gửi dữ liệu buffer1 theo giao thức văn bản
fwrite($client, $buffer1);
// Gửi dữ liệu buffer2 theo giao thức văn bản
fwrite($client, $buffer2);
```

Giao thức văn bản rất đơn giản và dễ sử dụng. Nếu nhà phát triển cần một giao thức riêng của mình, chẳng hạn như truyền dữ liệu với Ứng dụng di động hoặc giao tiếp với phần cứng, họ có thể xem xét sử dụng giao thức văn bản, cả phát triển và gỡ lỗi đều rất thuận tiện.

**Gỡ lỗi giao thức văn bản**

> Giao thức văn bản có thể được gỡ lỗi bằng cách sử dụng trình điều khiển telnet, ví dụ như dưới đây:

Tạo tệp test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

Chạy ```php test.php start``` sẽ hiển thị như sau

```txt
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Mở một cửa sổ terminal mới, sử dụng telnet để kiểm tra (đề xuất sử dụng telnet trên hệ điều hành Linux)

Giả sử là kiểm tra trên máy cục bộ,
Trên terminal thực hiện telnet 127.0.0.1 5678
Sau đó nhập hi và nhấn Enter
Sẽ nhận được dữ liệu hello world\n
```txt
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
