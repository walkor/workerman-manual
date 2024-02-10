# connect
**``` (Require Workerman version>=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Connect to Channel/Server

### Parameters
 ``` listen_ip ```

The IP address that Channel/Server listens to. Default is ```127.0.0.1``` if not passed.

 ``` listen_port ```
The port that Channel/Server listens to. Default is 2206 if not passed.

### Returns
void

### Example
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
