# onClose
## 說明:
```php
callback Worker::$onClose
```

當客戶端連接與Workerman斷開時觸發的回調函數。不管連接是如何斷開的，只要斷開就會觸發```onClose```。每個連接只會觸發一次```onClose```。

注意：如果對端是由於斷網或者斷電等極端情況斷開的連接，這時由於無法及時發送tcp的fin包給workerman，workerman就無法得知連接已經斷開，也就無法及時觸發```onClose```。這種情況需要通過應用層心跳來解決。workerman中連接的心跳實現參見[這裡](../faq/heartbeat.md)。如果使用的是GatewayWorker框架，則直接使用GatewayWorker框架的心跳機制即可，參見[這裡](https://doc2.workerman.net/heartbeat.html)。

由於udp是無連接的，所以當使用udp時不會觸發onConnect回調，也不會觸發onClose回調。

## 回調函數的參數

 ``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連接，如[發送數據](../tcp-connection/send.md)，[關閉連接](../tcp-connection/close.md)等


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其它回調寫法。
