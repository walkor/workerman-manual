# onError
## 說明:
```php
callback Worker::$onError
```

當客戶端的連線上發生錯誤時觸發。

目前錯誤類型有

1、調用Connection::send由於客戶端連線斷開導致的失敗（緊接著會觸發onClose回調）``` (code:WORKERMAN_SEND_FAIL msg:client closed)```

2、在觸發onBufferFull後(發送緩衝區已滿)，仍然調用Connection::send，並且發送緩衝區仍然是滿的狀態導致發送失敗(不會觸發onClose回調)``` (code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3、使用AsyncTcpConnection異步連接失敗時(緊接著會觸發onClose回調)``` (code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client返回的錯誤消息)```

## 回調函數的參數

 ``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連線，如[發送數據](../tcp-connection/send.md)，[關閉連線](../tcp-connection/close.md)等

 ``` $code ```

錯誤碼

 ``` $msg ```

錯誤消息

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其它回調寫法。
