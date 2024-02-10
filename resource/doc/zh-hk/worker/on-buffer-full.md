# onBufferFull
## 說明:
```php
callback Worker::$onBufferFull
```

每個連線都有一個獨立的應用層發送緩衝區，如果客戶端接收速度小於服務端發送速度，數據會在應用層緩衝區暫存，如果緩衝區滿則會觸發onBufferFull回調。

緩衝區大小為[TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)，默認值為1MB，可以為當前連線動態設置緩衝區大小，例如：
```php
// 設置當前連線發送緩衝區，單位字節
$connection->maxSendBufferSize = 102400;
```
也可以利用[TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) 設置所有連線默認緩衝區的大小，例如代碼：
```php
use Workerman\Connection\TcpConnection;
// 設置所有連線的默認應用層發送緩衝區大小，單位字節
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

該回調**可能**會在調用Connection::send後立即被觸發，例如發送大數據或者連續快速的向對端發送數據，由於網路等原因數據被大量積壓在對應連線的發送緩衝區，當超過```TcpConnection::$maxSendBufferSize```上限時觸發。

當發生onBufferFull事件時，開發者一般需要採取措施，例如停止向對端發送數據，等待發送緩衝區的數據被發送完畢(onBufferDrain事件)等。

當調用Connection::send(`$A`)後導致觸發onBufferFull時，不管本次send的數據`$A`多大，即使大於`TcpConnection::$maxSendBufferSize`，本次要發送的數據仍然會被放入發送緩衝區。也就是說發送緩衝區實際放入的數據可能遠遠大於`TcpConnection::$maxSendBufferSize`，當發送緩衝區的數據已經大於`TcpConnection::$maxSendBufferSize`時，仍然繼續Connection::send(`$B`)數據，則這次send的`$B`數據不會放入發送緩衝區，而是被丟棄掉，並觸發`onError`回調。

總結來說，只要發送緩衝區還沒滿，哪怕只有一個字節的空間，調用Connection::send(```$A```)肯定會把```$A```放入發送緩衝區，如果放入發送緩衝區後，發送緩衝區大小超過了```TcpConnection::$maxSendBufferSize```限制，則會觸發onBufferFull回調。

## 回調函數的參數

 ``` $connection ```

連線對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連線，如[發送數據](../tcp-connection/send.md)，[關閉連線](../tcp-connection/close.md)等

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
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其他回調寫法。

## 參見
onBufferDrain 當連線的應用層發送緩衝區數據全部發送完畢時觸發
