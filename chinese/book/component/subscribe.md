# subscribe

```php
void \Channel\Client::subscribe(string $subject)
```
订阅某个主题的消息

### 参数
``` subject ```

主题，类型为字符串


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
    Channel\Client::subscribe('subject_of_interest');
};

Worker::runAll();
```
