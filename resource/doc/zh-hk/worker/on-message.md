# onMessage
## 說明:
```php
callback Worker::$onMessage
```

當客戶端通過連接發來數據時(Workerman收到數據時)觸發的回調函數

## 回調函數的參數

 ``` $connection ```

連接對象，即[TcpConnection實例](../tcp-connection.md)，用於操作客戶端連接，如[發送數據](../tcp-connection/send.md)，[關閉連接](../tcp-connection/close.md)等

 ``` $data ```

客戶端連接上發來的數據，如果Worker指定了協議，則$data是對應協議decode（解碼）了的數據。數據類型與協議`decode()`實現有關，`websocket` `text` `frame` 為字符串，HTTP協議為[`Workerman\Protocols\Http\Request`](../http/request.md)對象。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// 運行worker
Worker::runAll();
```

提示：除了使用匿名函數作為回調，還可以[參考這裡](../faq/callback_methods.md)使用其它回調寫法。
