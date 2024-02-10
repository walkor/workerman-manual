# 連接
## 說明:
```php
array Worker::$connections
```

格式為
```php
array(id=>connection, id=>connection, ...)
```

此屬性中存儲了**當前進程**的所有的客戶端連接對象，其中id為connection的id編號，詳情見手冊[TcpConnection的id屬性](../tcp-connection/id.md)。

```$connections``` 在廣播時或者根據連接id獲得連接對象非常有用。

如果得知connection的編號為```$id```，可以很方便的通過```$worker->connections[$id]```獲得對應的connection對象，從而操作對應的socket連接，例如通過```$worker->connections[$id]->send('...')``` 發送數據。

注意：如果連接關閉(觸發onClose)，對應的```connection```會從```$connections```數組裡刪除。

注意：開發者不要對這個屬性做修改操作，否則可能造成不可預知的情況。


## 範例

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// 進程啟動時設置一個定時器，定時向所有客戶端連接發送數據
$worker->onWorkerStart = function($worker)
{
    // 定時，每10秒一次
    Timer::add(10, function()use($worker)
    {
        // 遍歷當前進程所有的客戶端連接，發送當前伺服器的時間
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// 運行worker
Worker::runAll();
```
