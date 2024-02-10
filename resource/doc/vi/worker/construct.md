# Hàm tạo __construct

## Giải thích:
```php
Worker::__construct([string $listen , array $context])
```

Khởi tạo một phiên bản của Worker container, có thể thiết lập một số thuộc tính và giao diện gọi lại của container, hoàn thành các chức năng cụ thể.

## Tham số
#### **``` $listen ```** (tham số tùy chọn, không điền có nghĩa không lắng nghe bất kỳ cổng nào)

Nếu thiết lập tham số lắng nghe ```$listen```, thì sẽ thực hiện lắng nghe socket.

Định dạng của $listen sẽ là <protocol>://<địa chỉ lắng nghe>

**<protocol> có thể là các định dạng sau:**

tcp: ví dụ ```tcp://0.0.0.0:8686```

udp: ví dụ ```udp://0.0.0.0:8686```

unix: ví dụ ```unix:///tmp/my_file ``` ```(yêu cầu Workerman>=3.2.7)```

http: ví dụ ```http://0.0.0.0:80```

websocket: ví dụ ```websocket://0.0.0.0:8686```

text: ví dụ ```text://0.0.0.0:8686``` ```(text là giao thức văn bản tích hợp trong Workerman, tương thích với telnet, xem chi tiết ở phần Text Protocol phụ lục)```

và các giao thức tùy chỉnh khác, xem phần tùy chỉnh giao tiếp trong tài liệu này.

**<địa chỉ lắng nghe> có thể có các định dạng sau:**

Nếu là unix socket, địa chỉ sẽ là đường dẫn cục bộ trên ổ đĩa.

Nếu không phải là unix socket, định dạng địa chỉ sẽ là <địa chỉ IP của máy>:<số cổng>

<địa chỉ IP của máy> có thể là ```0.0.0.0``` để lắng nghe trên tất cả các card mạng của máy, bao gồm cả IP nội và ngoại, cũng như loopback 127.0.0.1

<địa chỉ IP của máy> nếu là ```127.0.0.1``` thì chỉ lắng nghe loopback trong máy, chỉ máy đó mới có thể truy cập, bên ngoài không thể truy cập

Nếu <địa chỉ IP của máy> là IP nội, giống như ```192.168.xx.xx```, thì chỉ lắng nghe IP nội, người dùng bên ngoài không thể truy cập

Nếu giá trị của<địa chỉ IP của máy> không phải là IP của máy thì không thể lắng nghe và báo lỗi ```Cannot assign requested address```

**Lưu ý:** <số cổng> không thể lớn hơn 65535. Nếu <số cổng> nhỏ hơn 1024, cần quyền root để lắng nghe. Cổng lắng nghe phải là cổng chưa được sử dụng trên máy, nếu không sẽ không thể lắng nghe và báo lỗi ```Address already in use```

#### **``` $context ```**

Một mảng. Được sử dụng để truyền các tùy chọn ngữ cảnh của socket, xem [socket context options](https://php.net/manual/zh/context.socket.php)

## Ví dụ

Worker hoạt động như một container http lắng nghe và xử lý yêu cầu http
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Worker hoạt động như một container websocket lắng nghe và xử lý yêu cầu websocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Worker hoạt động như một container tcp lắng nghe và xử lý yêu cầu tcp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Worker hoạt động như một container udp lắng nghe và xử lý yêu cầu udp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Worker lắng nghe unix domain socket  ```(yêu cầu phiên bản Workerman>=3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Container Worker không lắng nghe bất kỳ cổng nào, chỉ để xử lý một số任何 nhiệm vụ định kỳ
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Thực hiện mỗi 2.5 giây
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Chạy worker
Worker::runAll();
```

**Worker lắng nghe cổng theo giao thức do người dùng định nghĩa**

Cấu trúc cuối cùng của thư mục
``` 
├── Protocols              // Đây là thư mục Protocols cần tạo
│   └── MyTextProtocol.php // Đây là tệp giao thức tùy chỉnh cần tạo
├── test.php  // Đây là tệp test cần tạo
└── Workerman // Thư mục mã nguồn Workerman, không được sửa mã nguồn bên trong
```

1. Tạo thư mục Protocols và tạo tệp giao thức
```Protocols/MyTextProtocol.php``` (xem cấu trúc thư mục ở trên)

```php
// namespace giao thức tùy chỉnh của người sử dụng được gán là Protocols
namespace Protocols;
// Giao thức văn bản đơn giản, định dạng giao thức là văn bản + dấu xuống dòng
class MyTextProtocol
{
    // Chức năng phân phối gói, trả về độ dài gói hiện tại
    public static function input($recv_buffer)
    {
        // Tìm dấu xuống dòng
        $pos = strpos($recv_buffer, "\n");
        // Không tìm thấy dấu xuống dòng, nghĩa là không phải là một gói hoàn chỉnh, trả về 0 và tiếp tục đợi dữ liệu
        if($pos === false)
        {
            return 0;
        }
        // Tìm thấy dấu xuống dòng, trả về độ dài gói hiện tại, bao gồm cả dấu xuống dòng
        return $pos+1;
    }

    // Sau khi nhận được một gói hoàn chỉnh, giải mã tự động thông qua decode, ở đây chỉ là loại bỏ dấu xuống dòng
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Trước khi gửi dữ liệu cho khách hàng, sẽ tự động mã hóa thông qua encode, sau đó mới gửi đến khách hàng, ở đây thêm dấu xuống dòng
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Sử dụng MyTextProtocol để lắng nghe và xử lý yêu cầu

Tạo tệp ```test.php``` theo cấu trúc thư mục ở trên

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Khi nhận được dữ liệu hoàn chỉnh (kết thúc bằng dấu Xuống dòng), tự động thực hiện MyTextProtocol::decode('dữ liệu nhận được')
 * Kết quả được truyền qua $data vào hàm gọi lại onMessage
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Gửi dữ liệu cho khách hàng, tự động gọi MyTextProtocol::encode('hello world') để mã hóa giao thức,
     * sau đó mới gửi đến khách hàng
     */
    $connection->send("hello world");
};

// Chạy tất cả các worker
Worker::runAll();
```

3. Kiểm tra

Mở terminal, đi đến thư mục chứa tệp ```test.php```, thực thi ```php test.php start```
``` 
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

Mở terminal và sử dụng telnet để kiểm tra (khuyến nghị sử dụng telnet trên hệ thống linux)

Giả sử là kiểm thử trên máy local,
Trong terminal, gõ telnet 127.0.0.1 5678
Sau đó gõ hi và Ấn Enter
Sẽ nhận được dữ liệu là hello world\n
``` 
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
