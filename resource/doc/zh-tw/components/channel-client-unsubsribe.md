# 取消訂閱

**```（要求Workerman版本>=3.3.0）```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```

取消訂閱某個事件，當該事件發生時，將不再觸發```on($event_name, $callback)```註冊的回調```$callback```

### 參數
 ``` $event_name ```

事件名稱

### 返回值
void



### 範例
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

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
