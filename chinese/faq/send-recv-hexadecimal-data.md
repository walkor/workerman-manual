# 如何发送接收16进制数据

**接收16进制数据**

当收到数据后用函数```bin2hex($data)```可以将数据转换成16进制。

**发送16进制数据**

发送数据前用```hex2bin($data)```将16进制数据转换成二进制发送。

**示例：**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 得到16进制数据
    $hex_data = bin2hex($data);
    // 向客户端发送16进制数据
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```



