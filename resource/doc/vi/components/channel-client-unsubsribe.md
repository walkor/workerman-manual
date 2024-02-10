# Hủy đăng ký

**``` (Yêu cầu Workerman phiên bản >=3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
Hủy đăng ký một sự kiện cụ thể, khi sự kiện này xảy ra, các callback đã đăng ký thông qua ```on($event_name, $callback)``` sẽ không còn được kích hoạt nữa.

### Tham số
 ``` $event_name ```

Tên của sự kiện

### Giá trị trả về
void

### Ví dụ
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
