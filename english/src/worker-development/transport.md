# transport
## Description:
```php
string Worker::$transport
```

Set the protocol of transport layer, tcp or udp.


## Examples

```php
use Workerman\Worker;
$worker = new Worker('Text://0.0.0.0:8484');
// udp
$worker->transport = 'udp';
$worker->onMessage = function($connection, $data)
{
    $connection->send('Hello');
};
```
