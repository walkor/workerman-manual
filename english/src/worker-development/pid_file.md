# pidFile
## Description:
```php
static Event Worker::$pidFile
```

This is a static property. ```Worker::$pidFile``` is a path which can be set to store the pid of the master process.


## Examples

```php
use Workerman\Worker;
Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('Text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
```
