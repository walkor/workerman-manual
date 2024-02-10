# 連接
**``` (要求Workerman版本>=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
連接Channel/Server

### 參數
 ``` listen_ip ```

Channel/Server 監聽的IP地址，不傳默認是```127.0.0.1```

 ``` listen_port ```

Channel/Server 監聽的端口，不傳默認是2206

### 返回值
void



### 示例
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
};

Worker::runAll();
```
