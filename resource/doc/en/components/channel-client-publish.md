# Publish
**``` (Requires Workerman version>=3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Publishes an event, and all subscribers to this event will receive it and trigger the callback ```$callback``` registered with ```on($event_name, $callback)```.

### Parameters
 ``` $event_name ```
The name of the event to be published, which can be any string. If there are no subscribers to the event, it will be ignored.

 ``` $event_data ```
The data related to the event, which can be a number, string, or array.

### Return
void

### Example
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
