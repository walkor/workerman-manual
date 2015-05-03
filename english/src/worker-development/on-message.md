# onMessage
## Description:
```php
callback Worker::$onMessage
```

Emitted when data is received.

## Parameters

``` $connection ```

The instance of Connection.

``` $data ```

The data received.

If the Protocol of Worker is setted, the data will be the result of ```Protocol::decode($recv_buffer)```.


## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // $data is the result of Websocket::decode($recv_buffer)
    var_dump($data);
    $connection->send('receive success');
};
```
