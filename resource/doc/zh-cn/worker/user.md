# user

## 说明:
```php
string Worker::$user
```

设置当前Worker实例以哪个用户运行。此属性只有当前用户为root时才能生效。不设置时默认以当前用户运行。

建议```$user```设置权限较低的用户，例如www-data、apache、nobody等。

注意：此属性必须在```Worker::runAll();```运行前设置才有效。windows系统不支持此特性。


## 范例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 设置实例的运行用户
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// 运行worker
Worker::runAll();
```
