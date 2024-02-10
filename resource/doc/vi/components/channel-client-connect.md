# kết nối
**``` (Yêu cầu phiên bản Workerman>=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Kết nối tới Channel/Server

### Tham số
``` listen_ip ```

Địa chỉ IP mà Channel/Server lắng nghe, nếu không truyền thì mặc định là```127.0.0.1```

``` listen_port ```

Cổng mà Channel/Server lắng nghe, nếu không truyền thì mặc định là 2206

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
};

Worker::runAll();
```
