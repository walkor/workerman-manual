# xuất bản
**```(Yêu cầu phiên bản Workerman>=3.3.0)```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Xuất bản một sự kiện nào đó, tất cả các người đăng ký sự kiện này sẽ nhận được sự kiện này và kích hoạt callback `$callback` đã đăng ký bằng `on($event_name, $callback)`

### Tham số
```$event_name```
Tên sự kiện được xuất bản, có thể là bất kỳ chuỗi nào. Nếu sự kiện không có người đăng ký, sự kiện sẽ bị bỏ qua.
```$event_data```
Dữ liệu liên quan đến sự kiện, có thể là số, chuỗi hoặc mảng

### Giá trị trả về
void

### Ví dụ
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
