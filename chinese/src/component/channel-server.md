# Channel组件服务端

**``` (要求Workerman版本>=3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

实例化一个\Channel\Server服务端

### 参数
``` listen_ip ```

监听的本机ip地址，不传默认是```0.0.0.0```

``` listen_port ```

监听的端口，不传默认是2206

## 例子

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
require_once './Channel/src/Server.php';

// 不传参数默认是监听0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

