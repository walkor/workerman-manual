# connections
## Description:
```php
array Worker::$connections
```

The format is 
```php
array(id=>connection, id=>connection, ...)
```

This property stores all the client connection objects of the **current process**, where id is the id number of the connection, details can be found in the manual [TcpConnection id property](../tcp-connection/id.md).

`$connections` is very useful when broadcasting or getting a connection object based on the connection id.

If the id of the connection is known as `$id`, it is very convenient to get the corresponding connection object through `$worker->connections[$id]` in order to operate the corresponding socket connection, for example, sending data through `$worker->connections[$id]->send('...')`.

Note: If the connection is closed (triggering onClose), the corresponding connection will be removed from the `$connections` array.

Note: Developers should not make modifications to this property, as it may cause unpredictable situations.


## Example

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Set a timer when the process starts to send data to all client connections at regular intervals
$worker->onWorkerStart = function($worker)
{
    // Every 10 seconds
    Timer::add(10, function()use($worker)
    {
        // Loop through all the client connections of the current process and send the current server time
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Run the worker
Worker::runAll();
```
