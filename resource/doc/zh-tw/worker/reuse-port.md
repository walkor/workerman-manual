# reusePort
> **注意**
> 需要workerman>= 3.2.1  PHP>=7.0，Windows系统及 Mac OS 不支持此特性

## 說明:

```php
bool Worker::$reusePort
```

設定當前worker是否開啟監聽端口複用（socket的SO_REUSEPORT選項）。

開啟監聽端口複用後，允許多個無親緣關係的進程監聽相同的端口，並且由系統內核做負載均衡，決定將socket連接交給哪個進程處理，避免了驚群效應，可以提升多進程短連接應用的性能。

**注意：** 此特性需要PHP版本>=7.0

## 範例 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// 運行worker
Worker::runAll();
```

## 範例2：workerman多端口(多協議)監聽
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// 每個進程啟動後在當前進程新增一個監聽
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * 多個進程監聽同一個端口（監聽套接字不是繼承自父進程）
     * 需要開啟端口複用，不然會報Address already in use錯誤
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 執行監聽
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// 運行worker
Worker::runAll();
```
