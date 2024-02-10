# onBufferFull
## 說明:
```php
callback Worker::$onBufferFull
```

每個連接都有一個獨立的應用層傳送緩衝區，如果客戶端接收速度小於服務端傳送速度，數據會在應用層緩衝區暫存，如果緩衝區滿則會觸發onBufferFull回調。

緩衝區大小為[TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)，默認值為1MB，可以為當前連接動態設置緩衝區大小例如：
```php
// 設置當前連接傳送緩衝區，單位字節
$connection->maxSendBufferSize = 102400;
```
也可以利用[TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) 設置所有連接預設緩衝區的大小，例如代碼：
```php
use Workerman\Connection\TcpConnection;
// 設置所有連接的預設應用層傳送緩衝區大小，單位字節
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

該回調**可能**會在調用Connection::send後立刻被觸發，例如傳送大數據或者連續快速的向對端傳送數據，由於網路等原因數據被大量積壓在對應連接的傳送緩衝區，當超過```TcpConnection::$maxSendBufferSize```上限時觸發。

當發生onBufferFull事件時，開發者一般需要採取措施，例如停止向對端傳送數據，等待傳送緩衝區的數據被傳送完畢(onBufferDrain事件)等。

當調用Connection::send(`$A`)後導致觸發onBufferFull時，不管本次send的數據`$A`多大，即使大於`TcpConnection::$maxSendBufferSize`，本次要傳送的數據仍然會被放入傳送緩衝區。也就是說傳送緩衝區實際放入的數據可能遠遠大於`TcpConnection::$maxSendBufferSize`，當傳送緩衝區的數據已經大於`TcpConnection::$maxSendBufferSize`時，仍然繼續Connection::send(`$B`)數據，則這次send的`$B`數據不會放入傳送緩衝區，而是被丟棄掉，並觸發`onError`回調。

總結來說，只要傳送緩衝區還沒滿，哪怕只有一個字節的空間，調用Connection::send(```$A```)肯定會把```$A```放入傳送緩衝區，如果放入傳送緩衝區後，傳送緩衝區大小超過了```TcpConnection::$maxSendBufferSize```限制，則會觸發onBufferFull回調。

## 回調函數的參數

``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連接，如[傳送數據](../tcp-connection/send.md)，[關閉連接](../tcp-connection/close.md)等

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
onBufferDrain 當連接的應用層傳送緩衝區數據全部傳送完畢時觸發
