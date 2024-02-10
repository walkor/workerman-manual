# 發佈
**```(要求Workerman版本>=3.3.0)```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```

發佈某個事件，所有訂閱此事件的用戶將接收到該事件，並觸發已註冊```on($event_name, $callback)```的回調```$callback```

### 參數
 ``` $event_name ```

發佈的事件名稱，可以是任意的字符串。如果事件沒有任何訂閱者，事件將被忽略。

 ``` $event_data ```

與事件相關的數據，可以是數字、字符串或數組。

### 返回值
void

### 示例
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
