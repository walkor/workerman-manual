# send
## Description:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Sends data on the connection.

If the protocol is setted, ```$data``` will be encoded with ```Protocol::encode($data)``` before send.

## Parameters

``` $data ```

The data to be sent.

``` $raw ```
Whether send raw data.  ```Protocol::encode($data)``` will not be called When  ```$raw=true```.

## Return Values

```true```: success

```null```: Join to send buffer and waiting to be sent asynchronously.

```false```: Connection is closed by remote or send buffer is full.


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    // hello\n Will be encode by \Workerman\Protocols\Websocket::encode before to be sent
    $connection->send("hello\n");
};

// Run all workers
Worker::runAll();
```
