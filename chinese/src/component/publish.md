# publish
**``` (要求Workerman版本>=3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
发布某个事件，所有这个事件的订阅者会收到这个事件并触发```on($event_name, $callback)```注册的回调```$callback```

### 参数
``` $event_name ```

发布的事件名称，可以是任意的字符串。如果事件没有任何订阅者，事件将被忽略。

``` $event_data ```

事件相关的数据，可以是数字、字符串或者数组

### 返回值
void



### 示例
```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
require_once './Channel/src/Client.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function($connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
