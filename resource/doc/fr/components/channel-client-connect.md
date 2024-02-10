# connect
**``` (Requires Workerman version >= 3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Connects to the Channel/Server.

### Parameters
``` listen_ip ```

The IP address that the Channel/Server listens on. If not provided, the default is ```127.0.0.1```.

``` listen_port ```

The port that the Channel/Server listens on. If not provided, the default is 2206.

### Return
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
