# stdoutFile

## Description
```php
static string Worker::$stdoutFile
```

This attribute is a global static attribute. If running in daemon mode (started with ```-d```), all output to the terminal (such as echo, var_dump, etc.) will be redirected to the file specified by stdoutFile.

If not set and running in daemon mode, all terminal output will be redirected to `/dev/null` (which means discarding all output by default).

> Note: `/dev/null` is a special file in Linux, which is actually a black hole, and all data written to this file will be discarded. If you don't want to discard the output, you can set `Worker::$stdoutFile` to a normal file path.

> Note: This attribute must be set before ```Worker::runAll();``` is called to take effect. This feature is not supported in Windows systems.

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// All printed output is saved in the /tmp/stdout.log file
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Run worker
Worker::runAll();
```
