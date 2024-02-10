# runAll
```php
void Worker::runAll(void)
```
Run all Worker instances.

**Note:**

Worker::runAll() will block permanently after execution, meaning that the code after Worker::runAll() will not be executed. All Worker instances should be instantiated before Worker::runAll().

### Parameters
No parameters

### Return Value
No return

## Example: Running Multiple Worker Instances

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Run all Worker instances
Worker::runAll();
```


**Note:**

The Windows version of Workerman does not support instantiating multiple Workers in the same file.
The example above cannot run on the Windows version of Workerman.

For the Windows version of Workerman, multiple Worker instances need to be initialized in different files, as shown below.

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// Run all Worker instances (only one instance here)
Worker::runAll();
```


start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Run all Worker instances (only one instance here)
Worker::runAll();
```
