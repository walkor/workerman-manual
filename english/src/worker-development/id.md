# id

## Description:
```php
int Connection::$id
```

In one worker process each new connection is given its own unique id, this id is stored in the id.


## Examples


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo $connection->id;
};

// Run all workers
Worker::runAll();
```
