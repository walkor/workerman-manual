# Protocol
Requirement```(workerman >= 3.2.7)```

## Description:
```php
string Worker::$protocol
```
Set the protocol class for the current Worker instance.

Note: The protocol processing class can be directly specified when initializing the Worker to listen. For example:
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

The above code is equivalent to the following code:


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * First, it will check if the user has a custom \Protocols\Http protocol class,
 * if not, use the built-in protocol class Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```
