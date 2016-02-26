# unsubscribe
**``` (要求Workerman版本>=3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
取消订阅某个事件，这个事件发生时将不会再触发```on($event_name, $callback)```注册的回调```$callback```

### 参数
``` $event_name ```

事件名称

### 返回值
void



### 示例
```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
require_once './Channel/src/Client.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
