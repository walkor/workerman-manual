# unsubscribe
**``` (Requires Workerman version >= 3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
Unsubscribe from a specific event, so that the callbacks registered using ```on($event_name, $callback)``` will not be triggered when this event occurs.

### Parameters
 ``` $event_name ```

The name of the event.

### Return Value
void

### Example
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
