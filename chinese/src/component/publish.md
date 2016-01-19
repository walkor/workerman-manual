# publish

```php
void \Channel\Client::publish(string $subject, mixed $message)
```
发布某个主题的消息，所有这个主题的订阅者会收到这条消息

### 参数
``` subject ```

主题

``` message ```

消息内容，类型可以是数组或者字符串

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
    $message = array(1,2,3);
    Channel\Client::publish('subject_of_interest', $message);
};

Worker::runAll();
```
