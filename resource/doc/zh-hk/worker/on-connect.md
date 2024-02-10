# onConnect
## 說明:
```php
callback Worker::$onConnect
```

當客戶端與 Workerman 建立連接時（TCP 三次握手完成後）觸發的回調函數。每個連接只會觸發一次```onConnect```回調。

注意：onConnect 事件僅僅代表客戶端與 Workerman 完成了 TCP 三次握手，這時客戶端還沒有發來任何數據，此時除了通過```$connection->getRemoteIp()```獲得對方 IP，沒有其他可以鑑別客戶端的數據或者信息，所以在 onConnect 事件裡無法確認對方是誰。要想知道對方是誰，需要客戶端發送鑑權數據，例如某個 token 或者用戶名密碼之類，在 [onMessage 回調](on-message.md)裡做鑑權。

由於 UDP 是無連接的，所以當使用 UDP 時不會觸發 onConnect 回調，也不會觸發 onClose 回調。

## 回調函數的參數
``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連接，如[發送數據](../tcp-connection/send.md)，[關閉連接](../tcp-connection/close.md)等

## 範例
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "新的連接來自 IP " . $connection->getRemoteIp() . "\n";
};
// 運行 worker
Worker::runAll();
```
提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其他回調寫法。
