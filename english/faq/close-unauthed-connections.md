# Close unauthenticated connections
**Question:**

How to close a client connection that has not sent any data within a specified time, for example, automatically close the client connection if no data is received within 30 seconds, in order to enforce that unauthenticated connections must be authenticated within a specified time?

**Answer:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Temporary add an auth_timer_id property to the $connection object to store the timer id
    // Close the connection after 30 seconds, the client needs to send authentication within 30 seconds to remove the timer
    $connection->auth_timer_id = Timer::add(30, function() use ($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch ($msg['type'])
    {
        case 'login':
            // Validation successful, delete the timer to prevent the connection from being closed
            Timer::del($connection->auth_timer_id);
            break;
        // Other cases
    }
    // Other actions
}
```
