# stdoutFile
## Description:
```php
static string Worker::$stdoutFile
```

This is a static property. All output (echo var_dump, etc.) to the terminal  will be redirected to the specified file when workerman run as **daemon** mode. The default value is ```/dev/null```.

```Worker::$stdoutFile``` only work in daemon mode. All output will redirected to terminal when in debug mode .


## Examples

```php
use Workerman\Worker;
Worker::$daemonize = true;

// All output will be redirected to /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('Text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
```
