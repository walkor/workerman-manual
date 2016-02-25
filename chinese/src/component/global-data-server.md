# GlobalData 组件服务端
**``` (要求Workerman版本>=3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

实例化一个\GlobalData\Server服务

### 参数
``` listen_ip ```

监听的本机ip地址，不传默认是```0.0.0.0```

``` listen_port ```

监听的端口，不传默认是2207


## 例子
```php
use Workerman\Worker;
require_once __DIR__ . '/../../Workerman/Autoloader.php';
require_once __DIR__ . '/../src/Server.php';

// 监听端口
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
