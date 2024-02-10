# logFile
## Description:
```php
static string Worker::$logFile
```
Used to specify the location of the workerman log file.

This file records workerman's own related logs, including startup, shutdown, and so on.

If not set, the file name defaults to workerman.log, and the file location is in the parent directory of Workerman.

**Note:**

This log file only records workerman's own related startup, shutdown, and other logs, and does not include any business logs.

Business log classes can be implemented using functions such as [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) or [error_log](https://php.net/manual/zh/function.error-log.php).

## Example:

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Run worker
Worker::runAll();
```
