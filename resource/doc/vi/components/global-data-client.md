# Client của thành phần GlobalData
**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Khởi tạo một đối tượng client \GlobalData\Client. Dữ liệu chia sẻ giữa các tiến trình thông qua việc đặt giá trị thuộc tính trên đối tượng client.

### Tham số
Địa chỉ máy chủ GlobalData, định dạng `<địa chỉ IP>:<cổng>`, ví dụ: `127.0.0.1:2207`.

Nếu là một cụm máy chủ GlobalData, chuyển một mảng địa chỉ, ví dụ: `array('10.0.0.10:2207', '10.0.0.0.11:2207')`.

## Giải thích
Hỗ trợ thực hiện các hoạt động gán giá trị, đọc giá trị, kiểm tra isset, và xóa giá trị.
Đồng thời hỗ trợ thao tác nguyên tử cas.

## Ví dụ

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Máy chủ GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Khi tiến trình khởi động
$worker->onWorkerStart = function()
{
    // Khởi tạo một client global data toàn cầu
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Mỗi khi máy chủ nhận được tin nhắn
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Thay đổi giá trị của $global->somedata, các tiến trình khác sẽ chia sẻ biến $global->somedata này
    global $global;
    echo "bây giờ global->somedata=".var_export($global->somedata, true)."\n";
    echo "đặt \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Tất cả các cách sử dụng (cũng có thể sử dụng trong môi trường php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## Lưu ý:
Thành phần GlobalData không thể chia sẻ dữ liệu loại tài nguyên như kết nối mysql, kết nối socket, v.v.

Nếu sử dụng GlobalData/Client trong môi trường Workerman, hãy khởi tạo đối tượng GlobalData/Client trong các lệnh gọi callback onXXX, ví dụ trong onWorkerStart.

Không thể thực hiện các thao tác chia sẻ biến như sau.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Có thể thực hiện như sau
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
