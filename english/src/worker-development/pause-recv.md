# pauseRecv
## Description:
```php
void Connection::pauseRecv(void)
```

Pauses the reading of data. That is, 'data' events will not be emitted. Useful to throttle back an upload.

## Parameters

This function has no parameters.

## Return Values
No value is returned.


## Examples

```php
use Workerman\Worker;
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
    }
};
```
