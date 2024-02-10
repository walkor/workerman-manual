# id

## 說明:
```php
int Connection::$id
```

連接的ID。這是一個自增的整數。

注意：workerman是多進程的，每個進程內部會維護一個自增的connection id，所以多個進程之間的connection id會有重複。
如果想要不重複的connection id，可以根據需要給connection->id重新賦值，例如加上worker->id前綴。

## 參見
[Worker的connections屬性](../worker/connections.md)


## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// 運行worker
Worker::runAll();
```
