# Dịch vụ máy chủ của thành phần GlobalData
**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Khởi tạo dịch vụ \GlobalData\Server

### Tham số
 ``` listen_ip ```

Địa chỉ IP của máy chủ lắng nghe, nếu không truyền tham số, mặc định là ```0.0.0.0```

 ``` listen_port ```

Cổng lắng nghe, nếu không truyền tham số, mặc định là 2207


## Ví dụ
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Lắng nghe cổng
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
