# GlobalData 組件伺服器端
**``` (要求 Workerman 版本>=3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

實例化一個\GlobalData\Server服務

### 參數
 ``` listen_ip ```

監聽的本機ip地址，不傳預設是```0.0.0.0```

 ``` listen_port ```

監聽的埠口，不傳預設是2207


## 例子
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 監聽埠口
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
