# Channel組件伺服器端

**```(要求Workerman版本>=3.3.0)```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

實例化一個\Channel\Server伺服器端

### 參數
 ``` listen_ip ```

監聽的本機ip地址，不傳默認是```0.0.0.0```

 ``` listen_port ```

監聽的端口，不傳默認是2206

## 例子

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 不傳參數默認是監聽0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
