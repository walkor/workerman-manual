# connect
**``` (要求Workerman版本>=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
连接Channel/Server

### 参数
``` listen_ip ```

Channel/Server 监听的ip地址，不传默认是```127.0.0.1```

``` listen_port ```

Channel/Server监听的端口，不传默认是2206

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
};

Worker::runAll();
```
