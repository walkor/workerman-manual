# getRemoteIp
## 说明:
```php
string Connection::getRemoteIp()
```

获得该连接的客户端ip

## 参数

无参数


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// 运行worker
Worker::runAll();
```
