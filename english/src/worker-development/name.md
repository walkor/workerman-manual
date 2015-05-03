# name

## Description:
```php
string Worker::$name
```

Set the name of worker, useful for status command.


## Examples
yourfile.php

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8485');
// set the worker name
$worker->name = 'MyWebsocketWorker';
$worker->count = 6;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};

// Run all workers
Worker::runAll();
```

```php yourfile.php start -d``` and ```php yourfile.php status``` you will see

```shell
php start.php status
Workerman[start.php] status
---------------------------------------GLOBAL STATUS--------------------------------------------
Workerman version:3.1.4          PHP version:5.4.37
start time:2015-05-03 15:03:59   run 0 days 0 hours
load average: 0, 0, 0
1 workers       6 processes
worker_name       exit_status     exit_count
MyWebsocketWorker 0                0
---------------------------------------PROCESS STATUS-------------------------------------------
pid     memory  listening                worker_name       connections total_request send_fail throw_exception
14773   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
14774   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
14775   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
14776   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
14777   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
14778   0.56M   websocket://0.0.0.0:8485 MyWebsocketWorker 0           0              0         0
```
