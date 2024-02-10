# Worker

## Description:
```php
string Worker::$user
```

Set the user under which the current Worker instance runs. This property only takes effect when the current user is root. If not set, it runs under the current user by default.

It is recommended to set `$user` to a low-privileged user, such as www-data, apache, nobody, etc.

Note: This property must be set before `Worker::runAll();` is called to take effect. This feature is not supported on Windows systems.

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set the user for the instance
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Run the worker
Worker::runAll();
```
