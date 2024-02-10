# onBufferDrain
## 說明:
```php
callback Worker::$onBufferDrain
```

每個連線都有一個獨立的應用層發送緩衝區，緩衝區大小由```TcpConnection::$maxSendBufferSize```決定，默認值為1MB，可以手動設置更改大小，更改後會對所有連線生效。

該回調在應用層發送緩衝區數據全部發送完畢後觸發。一般與onBufferFull配合使用，例如在onBufferFull時停止向對端繼續send數據，在onBufferDrain恢復寫入數據。



## 回調函數的參數
 ``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連接，如[發送數據](../tcp-connection/send.md)，[關閉連接](../tcp-connection/close.md)等


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其它回調寫法。

## 參見
onBufferFull 當連接的應用層發送緩衝區滿時觸發
