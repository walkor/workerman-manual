# Đối tượng máy chủ kênh (Channel)  

**``` (Yêu cầu phiên bản Workerman >= 3.3.0)```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Khởi tạo một máy chủ \Channel\Server

### Tham số
 ``` listen_ip ```

Địa chỉ IP của máy chủ lắng nghe, mặc định là ```0.0.0.0``` nếu không truyền giá trị

 ``` listen_port ```

Cổng lắng nghe, mặc định là 2206 nếu không truyền giá trị

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Nếu không truyền tham số, mặc định là lắng nghe trên 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
