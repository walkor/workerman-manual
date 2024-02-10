# Basic Debugging

WorkerMan has two running modes, debug mode and daemon mode.

Run ```php start.php start``` to enter debug mode, where functions like ```echo, var_dump, var_export``` in the code will be displayed in the terminal. Note that when running WorkerMan with ```php start.php start```, all processes will exit when the terminal is closed.

On the other hand, running ```php start.php start -d``` enters daemon mode, which is the formal online running mode and is not affected by closing the terminal.

If you want to see the output of functions like ```echo, var_dump, var_export``` even when running in daemon mode, you can set the Worker::$stdoutFile property, for example:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Redirect screen output to the file specified by Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

This way, all the output from functions like ```echo, var_dump, var_export``` will be written to the file specified by ```Worker::$stdoutFile```. Note that the path specified by ```Worker::$stdoutFile``` must have write permissions.
