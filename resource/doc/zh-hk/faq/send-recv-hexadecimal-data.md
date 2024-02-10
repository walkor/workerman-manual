# 如何發送接收16進位數據

**接收16進位數據**

當收到數據後，可以使用函數```bin2hex($data)```將數據轉換成16進位。

**發送16進位數據**

在發送數據前，使用```hex2bin($data)```將16進位數據轉換成二進位後發送。

**範例：**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 得到16進位數據
    $hex_data = bin2hex($data);
    // 向客戶端發送16進位數據
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
