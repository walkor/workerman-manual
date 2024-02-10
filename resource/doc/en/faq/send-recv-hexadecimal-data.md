# How to Send and Receive Hexadecimal Data

**Receiving Hexadecimal Data**

Upon receiving data, the function `bin2hex($data)` can be used to convert the data into hexadecimal format.

**Sending Hexadecimal Data**

Before sending data, use `hex2bin($data)` to convert the hexadecimal data into binary for transmission.

**Example:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Get the hexadecimal data
    $hex_data = bin2hex($data);
    // Send hexadecimal data to the client
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
