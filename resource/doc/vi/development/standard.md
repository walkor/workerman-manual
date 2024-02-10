# Quy tắc phát triển

## Thư mục ứng dụng

Thư mục ứng dụng có thể được đặt ở bất kỳ vị trí nào.

## Tệp đầu vào

Tương tự như ứng dụng PHP trong môi trường nginx+PHP-FPM, ứng dụng trong WorkerMan cũng cần một tệp đầu vào, tên tệp đầu vào không có yêu cầu cụ thể, và tệp này chạy bằng cách sử dụng PHP Cli.

Trong tệp đầu vào chứa mã để tạo các quá trình lắng nghe, ví dụ đoạn mã dưới đây dựa trên Worker để phát triển:

test.php 
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Tạo một Worker lắng nghe cổng 2345, sử dụng giao thức http
$http_worker = new Worker("http://0.0.0.0:2345");

// Khởi động 4 quá trình để cung cấp dịch vụ bên ngoài
$http_worker->count = 4;

// Khi nhận được dữ liệu từ trình duyệt, gửi chuỗi "hello world" đến trình duyệt
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Gửi chuỗi "hello world" đến trình duyệt
    $connection->send('hello world');
};

Worker::runAll();
```

## Quy tắc mã trong WorkerMan

1. Tên lớp sử dụng đúng quy tắc viết hoa chữ cái đầu, tên tệp lớp phải giống với tên lớp bên trong tệp, để có thể tải tự động. Ví dụ:

```php
class UserInfo
{
...
```

2. Sử dụng không gian tên, tên không gian tương ứng với đường dẫn thư mục và dựa trên thư mục gốc của dự án của nhà phát triển.

Ví dụ, dự án là MyApp/, tệp lớp là MyApp/MyClass.php vì nó nằm trong thư mục gốc của dự án, nên không gian tên được bỏ qua. Tệp lớp MyApp/Protocols/MyProtocol.php vì MyProtocol.php nằm trong thư mục Protocols của dự án MyApp, nên cần thêm không gian tên `namespace Protocols;`. Đoạn mã như sau:

```php
namespace Protocols;
class MyProtocol
{
....
```

3. Tên hàm và biến thông thường sử dụng kiểu viết thường và gạch dưới, ví dụ:

```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Thành viên của lớp và phương thức lớp sử dụng kiểu viết thường chữ cái đầu dạng lạc đà, ví dụ:

```php
public $connectionList;
public function getConnectionList();
```

5. Tham số của hàm và các tham số của lớp sử dụng kiểu viết thường và gạch dưới, ví dụ:

```php
function get_connection_list($one_param, $tow_param)
{
....
```
