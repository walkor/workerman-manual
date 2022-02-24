# resumeRecv
## Description:
```php
void Connection::resumeRecv(void)
```

Resumes reading after a call to pause().

## Parameters

This function has no parameters.

## Return Values

No value is returned.

## Examples

```php
use Workerman\Worker;
use Workerman\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    $connection->messageCount = 0;
};
$worker->onMessage = function($connection, $data)
{
    // Stop receive data when 100 package received
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Resumes reading after 30s
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};

// Run all workers
Worker::runAll();
```
