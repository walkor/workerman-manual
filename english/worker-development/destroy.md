# destroy
## Description:
```php
void Connection::destroy()
```

Destroy the connection.

```$connection->onClose``` event will be called directly following this event

## Parameters

This function has no parameters.

## Return Values

No value is returned.

## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // if something wrong
    $connection->destroy();
};

// Run all workers
Worker::runAll();
```
