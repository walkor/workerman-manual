# 如何發送接收16進制數據

**接收16進制數據**

當收到數據後，可以使用```bin2hex($data)```函數將數據轉換為16進制。

**發送16進制數據**

在發送數據之前，使用```hex2bin($data)```將16進制數據轉換為二進制後再發送。

**示例：**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 獲得16進制數據
    $hex_data = bin2hex($data);
    // 向客戶端發送16進制數據
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
