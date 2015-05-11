# id

## 说明:
```php
int Connection::$id
```

连接的id。这是一个自增的整数


## 范例


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo $connection->id;
};
// 运行worker
Worker::runAll();
```
